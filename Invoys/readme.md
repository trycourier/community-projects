# How to Send Invoice and Add Payment Reminder in Next.js with Courier API

## Background
On the internet, a lot of open-source Invoice management apps are built with Laravel. As a Javascript developer, I want to build the “React Solution” for devs that are familiar with React and Javascript.

A problem I found when building with services in node.js is there is no built-in mailer. So, I have to search for a 3rd party service to do that for me. In this article, I will be integrating [Courier](https://www.courier.com/docs/) to send emails for this project [https://github.com/fazzaamiarso/invoys](https://github.com/fazzaamiarso/invoys).  

## Pre-requisites
As this article isn't your typical follow-along (more like "please sit tight and see how I do it"), it's not mandatory to be familiar with all technologies used. However, familiarity with Typescript and Next.js will be beneficial for quicker understanding. 

Techs in this blog:
- [Typescript](https://www.typescriptlang.org/). Type-safety and auto-completion are the best, right? 
- [Next.js](https://nextjs.org/). A production-ready framework to build a full-stack app, even for beginners.
- [Prisma](https://www.prisma.io/). A great ORM to work with databases. We use Prisma because of its type-safety and auto-completion, providing great developer experience with typescript added.
- [Trpc](https://trpc.io/). Enable us to easily build end-to-end type-safety between our Next.js client and server.
- Courier API. A great service/platform to handle our notifications, such as email, SMS, and much more.

You can find the full [source code here](https://github.com/fazzaamiarso/invoys/tree/bc231301c92bb07692a3388bb50d76d61603a41f) for reference.

### Goals
Before building the features, let's define our goals.
1. Send invoice link to client's email.
2. Send a reminder a day before an invoice's due date.
3. Cancel an invoice due date reminder when the invoice is already paid.
4. Handling network errors.

### Part 1: Setup Courier Platform
Let's head to Courier Dashboard. By default, it's in a production environment. Since I want to test things out, I'm going to change to the test environment by clicking the dropdown in the top-right corner. 

> We can copy all templates later to production or vice-versa.

Now, I will create a [**brand**](https://www.courier.com/docs/getting-started/courier-concepts/#brands) for my email notifications.

![go to brand](https://i.imgur.com/1Us0dzC.gif)

I'm just going to add a logo (beware that the logo width is fixed to 140px) on the header and social links on the footer. The designer UI is pretty straightforward, so here is the final result. 

<img src="https://i.imgur.com/sFrofll.png" alt="brand template" width="500px" />

Don't forget to publish the changes.

### Part 2: Send Invoice to Email
Currently, the send email button on the UI is doing nothing.

I'm going to create a `courier.ts` file in `src/lib/` to keep all Courier-related code. Also, I will use [courier node.js client library](https://github.com/trycourier/courier-node) which already abstracted all Courier API endpoints to functions.

Before I code the functionality, let's create the email notification design in courier designer and set up a Gmail provider.

On the email designer page, We will see that the created brand is already integrated. After that, let's design the template accordingly with the needed data. Here is the final result.

<img src="https://i.imgur.com/qFhHvAP.png" alt="email template final" width="500px" />

<img src="https://i.imgur.com/B6vKjpH.png" alt="action button" width="500px" />

Notice the value with `{}` that becomes green, it means it's a variable that can be inserted dynamically. I also set the 'See Invoice' button (or action) with a variable.

Before I can use the template, I need to create a *test event* by clicking the preview tab. Then, it will show a prompt to name the event and set `data` in JSON format. That data field is what will populate the value of the green `{}` variables (the data can be set from code also). Since it's a test event, I will fill it with arbitrary values.

Next, I publish the template so I can use it. Then, go to send tab. It will show the necessary code to send the email programmatically and the `data` will be populated with the previous *test event* that I created. 

<img src="https://i.imgur.com/PMGcVQq.png" alt="code snippet" width="500px" />

#### Backend
I will copy the test `AUTH_TOKEN` to the `.env` file and copy the snippet to `src/lib/courier.ts`.
```ts
const authToken = process.env.COURIER_AUTH_TOKEN;

// email to receive all sent notifications in DEVELOPMENT mode
const testEmail = process.env.COURIER_TEST_EMAIL;

const INVOICE_TEMPLATE_ID = <TEMPLATE_ID>;

const courierClient = CourierClient({
  authorizationToken: authToken,
});
```

Create a `sendInvoice` function that will be responsible for sending an email. To send an email from the code, I use the `courierClient.send()` function.
```ts
// src/lib/courier.ts

export const sendInvoice = async ({
  customerName,
  invoiceNumber,
  invoiceViewUrl,
  emailTo,
  productName,
  dueDate,
}: SendInvoice) => {

const recipientEmail = process.env.NODE_ENV === "production" ? emailTo : testEmail;
   
const { requestId } = await courierClient.send({
      message: {
        to: {
          email: recipientEmail,
        },
        template: INVOICE_TEMPLATE_ID,
        // Data for courier template designer
        data: {
          customerName,
          invoiceNumber,
          invoiceViewUrl,
          productName,
          dueDate,
        },
      },
    });
    return requestId
};
```

Define types for the `sendInvoice` function.
```ts
// src/lib/courier.ts

interface SendInvoice {
  productName: string;
  dueDate: string;
  customerName: string;
  invoiceNumber: string;
  invoiceViewUrl: string;
  emailTo: string;
}
```

Now that I can send the email, I will call it in the `sendEmail` trpc endpoint that resides in `src/server/trpc/router/invoice.ts`.

> Just remember that trpc endpoint is a Next.js API route. In this case, `sendEmail` will be the same as calling the `/api/trpc/sendEmail` route with `fetch` under the hood. For more explanation [https://trpc.io/docs/quickstart](https://trpc.io/docs/quickstart).

```ts
// src/server/trpc/router/invoice.ts
import { sendInvoice } from '@lib/courier';
import { dayjs } from '@lib/dayjs';

// .....SOMEWHERE BELOW
  sendEmail: protectedProcedure
    .input(
      z.object({
        customerName: z.string(),
        invoiceNumber: z.string(),
        invoiceViewUrl: z.string(),
        emailTo: z.string(),
        invoiceId: z.string(),
        productName: z.string(),
        dueDate: z.date(),
      })
    )
    .mutation(async ({ input }) => {
      const invoiceData = {
        ...input,
        dueDate: dayjs(input.dueDate).format('D MMMM YYYY'),
      };

      await sendInvoice(invoiceData);
    }),
```
For those who are unfamiliar with trpc, What I did is the same as handling a `POST` request. Let's break it down.

1. Trpc way of defining **request input from client** by validating with Zod. Here I define all data that are needed for the `sendInvoice` function.
```ts
.input(
      z.object({
        customerName: z.string(),
        invoiceNumber: z.string(),
        invoiceViewUrl: z.string(),
        emailTo: z.string(),
        invoiceId: z.string(),
        productName: z.string(),
        dueDate: z.date(),
      })
    )
```
2. Define a `POST` request handler (mutation).
```ts
// input from before
 .mutation(async ({ input }) => {
      const invoiceData = {
        ...input,
        // format a date to string with a defined format. 
        dueDate: dayjs(input.dueDate).format('D MMMM YYYY'), // ex.'2 January 2023'
      };

      // send the email
      await sendInvoice(invoiceData);
    }),
```

#### Frontend
Now, I can start to add the functionality to the send email button. I'm going to use the `trpc.useMutation()` function which is a thin wrapper of `tanstack-query's `useMutation`.

Let's add the mutation function. On successful response, I want to send a success toast on UI.
```tsx
//src/pages/invoices/[invoiceId]/index.tsx
import toast from 'react-hot-toast';

const InvoiceDetail: NextPage = () => {
// calling the `sendEmail` trpc endpoint with tanstack-query.
  const sendEmailMutation = trpc.invoice.sendEmail.useMutation({
    onSuccess() {
      toast.success('Email sent!');
    }
  });

}
```
I can just use the function as an inline handler, but I want to create a new handler for the button.
```tsx
//src/pages/invoices/[invoiceId]/index.tsx
 
 // still inside the InvoiceDetail component
 const sendInvoiceEmail = () => {
    const hostUrl = window.location.origin;

   // prevent a user from spamming when the API call is not done.
    if (sendEmailMutation.isLoading) return;

    // send input data to `sendEmail` trpc endpoint
    sendEmailMutation.mutate({
      customerName: invoiceDetail.customer.name,
      invoiceNumber: `#${invoiceDetail.invoiceNumber}`,
      invoiceViewUrl: `${hostUrl}/invoices/${invoiceDetail.id}/preview`,
      emailTo: invoiceDetail.customer.email,
      invoiceId: invoiceDetail.id,
      dueDate: invoiceDetail.dueDate,
      productName: invoiceDetail.name,
    });
  };
```
Now I can attach the handler to the send email button.
```tsx
//src/pages/invoices/[invoiceId]/index.tsx

<Button
   variant="primary"
   onClick={sendInvoiceEmail}
   isLoading={sendEmailMutation.isLoading}>
   Send to Email
</Button>
```

Here's the working UI.

<img src="https://i.imgur.com/ush8wdI.gif" alt="working ui" width="500px" />

### Part 3: Send Payment Reminder
I want to schedule a reminder that will be sent a day before an invoice's due date. To do that I'm going to use [Courier Automation API](https://www.courier.com/docs/automations/). 

First, let's design the email template in Courier designer. As I already go through the process before, here is the final result.

<img src="https://i.imgur.com/4l6shqK.png" alt="payment reminder template" width="500px" />

Before adding the function, define the types for the parameter and refactor the types.
```tsx
// src/lib/courier

interface CourierBaseData {
  customerName: string;
  invoiceNumber: string;
  invoiceViewUrl: string;
  emailTo: string;
}

interface SendInvoice extends CourierBaseData {
  productName: string;
  dueDate: string;
}

interface ScheduleReminder extends CourierBaseData {
  scheduledDate: Date;
  invoiceId: string;
}
```
Now, I add the `scheduleReminder` function to `src/lib/courier`
```tsx
//src/pages/invoices/[invoiceId]/index.tsx

// check if the development environment is production
const __IS_PROD__ = process.env.NODE_ENV === 'production';

const PAYMENT_REMINDER_TEMPLATE_ID = '<TEMPLATE_ID>';

export const scheduleReminder = async ({
  scheduledDate,
  emailTo,
  invoiceViewUrl,
  invoiceId,
  customerName,
  invoiceNumber,
}: ScheduleReminder) => {
  
  // delay until a day before due date in production, else 20 seconds after sent for development
  const delayUntilDate = __IS_PROD__
    ? scheduledDate
    : new Date(Date.now() + SECOND_TO_MS * 20);

  const recipientEmail = __IS_PROD__ ? emailTo : testEmail;
   
   // define the  automation steps programmatically
   const { runId } = await courierClient.automations.invokeAdHocAutomation({
     automation: {
       steps: [
         // 1. Set delay for the next steps until given date in ISO string
          { action: 'delay', until: delayUntilDate.toISOString() },

          // 2. Send the email notification. Equivalent to `courierClient.send()`
          {
            action: 'send',
            message: {
              to: { email: recipientEmail },
              template: PAYMENT_REMINDER_TEMPLATE_ID,
              data: {
                invoiceViewUrl,
                customerName,
                invoiceNumber,
              },
            },
          },
        ],
      },
    });

   return runId;
};
```
To send the reminder, I will call `scheduleReminder` after a successful `sendInvoice` attempt. Let's modify the `sendEmail` trpc endpoint.
```tsx
// src/server/trpc/router/invoice.ts

sendEmail: protectedProcedure
    .input(..) // omitted for brevity
    .mutation(async ({ input }) => {
      // multiplier for converting day to milliseconds.
      const DAY_TO_MS = 1000 * 60 * 60 * 24;

      // get a day before the due date
      const scheduledDate = new Date(input.dueDate.getTime() - DAY_TO_MS * 1);

      const invoiceData = {..}; //omitted for brevity

      await sendInvoice(invoiceData);

      //after the invoice is sent, schedule the reminder
      await scheduleReminder({
        ...invoiceData,
            scheduledDate,
      });
    }
```
Now if I try to send an invoice by email, I should get a reminder 20 seconds later since I'm in the development environment.

<img src="https://i.imgur.com/MfwQ6F0.png" alt="with payment reminder" />

### Part 4: Cancel a reminder
Finally, all the features are ready. However, I got a problem, what if a client had paid before the scheduled date for payment reminder? currently, the reminder email will still be sent. That's not a great user experience and potentially a confused client. Thankfully, Courier has an automation cancellation feature.

Let's add `cancelAutomationWorkflow` function that can cancel any automation workflow in `src/lib/courier.ts`.
```ts
export const cancelAutomationWorkflow = async ({
  cancelation_token,
}: {
  cancelation_token: string;
}) => {
    const { runId } = await courierClient.automations.invokeAdHocAutomation({
      automation: {
        // define a cancel action, that sends a cancelation_token
        steps: [{ action: 'cancel', cancelation_token }],
      },
    });

   return runId;
};
``` 
What is a cancelation_token? It's a unique token that can be set to an automation workflow, so it's cancelable by sending a `cancel` action with a matching `cancelation_token`. 

Add cancelation_token to `scheduleReminder`, I use the invoice's Id as a token.
```ts
// src/lib/courier.ts

export const scheduleReminder = async (..) => {
  // ...omitted for brevity

  const { runId } = await courierClient.automations.invokeAdHocAutomation({
    automation: {
      // add cancelation token here
      cancelation_token: `${invoiceId}-reminder`,
      steps: [
        { action: 'delay', until: delayUntilDate.toISOString() },
  
  // ... omitted for brevity
``` 
I will call `cancelAutomationWorkflow` when an invoice's status is updated to `PAID` in the `updateStatus` trpc endpoint. 
```ts
// src/server/trpc/router/invoice.ts

 updateStatus: protectedProcedure
    .input(..) // omitted for brevity
    .mutation(async ({ ctx, input }) => {
      const { invoiceId, status } = input;

      // update an invoice's status in database
      const updatedInvoice = await ctx.prisma.invoice.update({
        where: { id: invoiceId },
        data: { status },
      });

      // cancel payment reminder automation workflow if the status is paid.
      if (updatedInvoice.status === 'PAID') {

        //call the cancel workflow to cancel the payment reminder for matching cancelation_token.
        await cancelAutomationWorkflow({
          cancelation_token: `${invoiceId}-reminder`,
        });
      }

      return updatedStatus;
    }),
```
Here is the working UI.

<img src="https://i.imgur.com/jnLk4Tg.gif" alt="cancel log" width="500px" />

### Part 5: Error Handling
An important note when doing network requests is there are possibilities of failed requests/errors. I want to handle the error by throwing it to the client, so it can be reflected in UI.

On error, Courier API throws an error with `CourierHttpClientError` type by default. I will also have all functions' return value in `src/lib/courier.ts` consistent with the below format.
```ts
// On Success
type SuccessResponse = { data: any, error: null }

// On Error
type ErrorResponse = { data: any, error: string }
``` 
Now, I can handle errors by adding a `try-catch` block to all functions in `src/lib/courier.ts`.
```ts
try {
  // ..function code
  
  // modified return example
  return { data: runId, error: null };

} catch (error) {
  // make sure it's an error from Courier
  if (error instanceof CourierHttpClientError) {
      return { data: error.data, error: error.message };
    } else {
      return { data: null, error: "Something went wrong!" };
    }
}
```
Let's see a handling example on the `sendEmail` trpc endpoint.
```ts
// src/server/trpc/router/invoice.ts

  const { error: sendError } = await sendInvoice(..);
  if (sendError) throw new TRPCClientError(sendError);

  const { error: scheduleError } = await scheduleReminder(..);
  if (scheduleError) throw new TRPCClientError(scheduleError);
```

### Part 6: Go To Production
Now that all templates are ready, I will copy all assets in the test environment to production. Here is an example.

<img src="https://i.imgur.com/Mr3bmuH.gif" alt="copy assets to production" width="500px" />

## Conclusion
Finally, all the features are integrated with Courier. We've gone through a workflow of integrating Courier API to a Next.js application. Although it's in Next.js and trpc, the workflow will be pretty much the same with any other technology. I hope now you can integrate Courier into your application by yourself.

## About the Author
I'm Fazza Razaq Amiarso, a full-stack web developer from Indonesia. I'm also an Open Source enthusiast. I love to share my knowledge and learning [on my blog](https://fazzaamiarso.me/). I occasionally help other developers on [FrontendMentor](https://www.frontendmentor.io/profile/fazzaamiarso) in my free time.

Connect with me on [LinkedIn](https://www.linkedin.com/in/fazzaamiarso/)

## Quick Links

🔗 [Courier Docs](https://www.courier.com/docs/)

🔗 [Contribute to Invoys](https://github.com/fazzaamiarso/invoys)

🔗 [Invoys Motivation](https://fazzaamiarso.me/projects/invoys)