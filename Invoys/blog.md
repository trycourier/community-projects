# How to Send Invoice and Add Payment Reminder in Next.js with Courier API

## Background
On the internet, a lot of open-source Invoice management apps are built with Laravel. As a Javascript developer, I want to build the “React Solution” for devs that are familiar with React and Javascript.

Tech Stack:
- Next.js + Typescript
- TRPC
- Prisma
- Tailwind CSS
- Courier API

## Pre-requisites
In this article, you will see my process of integrating Courier API to [this project](https://github.com/fazzaamiarso/invoys). You should be familiar with Typescript and Next.js. As for the other technology, familiar will be good but not mandatory.

### Goals
Before building the features, let's define our goals.
- can send invoice link to client's email.
- can send a reminder a day before an invoice's due date.
- can cancel an invoice due date reminder when the invoice is already paid.
- can schedule an overdue reminder by checking all pending invoices due date daily.

### Part 1: Setup Courier Platform
Let's head to Courier Dashboard. By default, it's in production environment. Since I want to test things out, I'm going to change to test environment. We can copy our template and design later to production.

[GIF OF SWITCHING ENV]

Now, I will create a [**brand**](https://www.courier.com/docs/getting-started/courier-concepts/#brands) for my email notifications.

[GIF OF CREATING NEW BRAND FROM HOME]

I'm just going to add a logo (beware that the logo width is fixed to 140px) on the header and some social links on footer. The UI is pretty straight-forward, so here is the final result. 

[THE BRAND FINAL RESULT]

Don't forget to publish it.

### Part 2: Send Invoice to Email
Currently the send email button is doing nothing.

[GIF OF CLICKING SEND EMAIL BUTTON DOING NOTHING]

I'm going to create `src/lib/courier.ts` file to keep all Courier related API call. Also, I will use the [courier node.js client library](https://github.com/trycourier/courier-node) that already abstracted all Courier API endpoint to a function.

Before I code the functionality, let's create the email notification design first in courier designer.

[GIF OF CREATING NEW EMAIL NOTIFICATION]

We can see that the created brand already integrated, neat. After that, let's modify the email accordingly with the data needed. Here is the final result.

[PIC OF FINAL EMAIL TEMPLATE RESULT]

Notice the value with `{}` that become green, it means it's a variable that we can inject dynamically. I'll explain that later. Before I can send anything, I need to create a test event by clicking preview. Then, it will show a prompt to name the event and set `data` in json. That data field is what will populate the value of the green `{]` variables. Since it's a test event, I will fill it with any value, but later I will fill it with dynamic data from code. 

[GIF OF CREATING TEST EVENT]

Now, I will publish the template so I can send it and go to send tab. It will show the code snippet to send the email, although need modification from my end. 

[SHOW COURIER SEND TAB PIC]

#### Backend
I will copy the test `AUTH_TOKEN` to `.env` file and copy the snippet to `src/lib/courier.ts`.
```ts
const authToken = process.env.COURIER_AUTH_TOKEN;

// email to receive all sent notification in DEVELOPMENT
const testEmail = process.env.COURIER_TEST_EMAIL;

const INVOICE_TEMPLATE_ID = <TEMPLATE_ID>;

const courierClient = CourierClient({
  authorizationToken: authToken,
});
```

Create `sendInvoice` function that will be responsible for sending email with Courier programmatically. To send an email from code, I use `courierClient.send()` function.
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
        // Data for courier template desginer
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

Now that I can send the email, I will call it in `sendEmail` trpc endpoint that resides in `src/server/trpc/router/invoice.ts`.

> Just remember that trpc endpoint is basically a Next.js API route. In this case, `sendEmail` will be the same as calling `/api/trpc/sendEmail` route with `fetch` under the hood. For more explanation [https://trpc.io/docs/quickstart](https://trpc.io/docs/quickstart).

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
For those who are unfamilliar with trpc, What I did is the same as handling `POST` request. Let's break it down.

1. Trpc way of defining **request input from client** by validating with zod. Here I define all datas that are needed for the `sendInvoice` function.
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
        // format a date to string with defined format. 
        dueDate: dayjs(input.dueDate).format('D MMMM YYYY'), // ex.'2 January 2023'
      };

      // send the email
      await sendInvoice(invoiceData);
    }),
```

#### Frontend
Now, I can start to add the functionality to the send email button. I'm going to use `trpc.useMutation()` function which is a thin abstraction layer on `tanstack-query`.

Let's add the mutation function. On sucessful response, I want to send a success toast on UI.
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
I can just use the function as handler, but I want to create a new handler for the button.
```tsx
//src/pages/invoices/[invoiceId]/index.tsx
 
 // still inside the InvoiceDetail component
 const sendInvoiceEmail = () => {
    const hostUrl = window.location.origin;

   // prevent user from spamming when API call is not done.
    if (sendEmailMutation.isLoading) return;

    // send the input to `sendEmail` trpc endpoint
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
   isLoading={sendEmailMutation.isLoading}
   Send to Email
</Button>
```

It should work.

[SHOW THE WORKING UI]

[CHECK EMAIL]


### Part 3: Send Payment Reminder
I want to schedule a reminder that will be sent a day before an invoice's due date. To do that I'm going to use Courier Automation API. 

But first, let's setup the email template design in Courier designer. As I already go through the process before, here is the final result.

[THE FINAL TEMPLATE DESIGN]

Before adding the function, define the types for the paramater and refactor the types.
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
Now, I add `scheduleReminder` function to `src/lib/courier`
```tsx
const PAYMENT_REMINDER_TEMPLATE_ID = '<TEMPLATE_ID>';

export const scheduleReminder = async ({
  scheduledDate,
  emailTo,
  invoiceViewUrl,
  invoiceId,
  customerName,
  invoiceNumber,
}: ScheduleReminder) => {

   // delay until a day before due in production else 20 seconds after sent for development
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
To send the reminder, I will call `scheduleReminder` after a successful `sendInvoice` attempt. Let's modify `sendEmail` trpc endpoint.
```tsx
// src/server/trpc/router/invoice.ts

sendEmail: protectedProcedure
    .input(..) // omitted for brevity
    .mutation(async ({ input }) => {
      const DAY_TO_MS = 1000 * 60 * 60 * 24;

      // get a day before due date
      const scheduledDate = new Date(input.dueDate.getTime() - DAY_TO_MS * 1);

      const invoiceData = {..}; //omitted for brevity

      await sendInvoice(invoiceData);

      //after the invoice sent, schedule the reminder
      await scheduleReminder({
            ...invoiceData,
            scheduledDate,
      });
    }
```
Now if we try to send invoice to email, I should get a reminder 20 seconds later since I'm in development environment.

[SHOW THE UPDATED UI]

#### Part 4: Cancel a reminder
Finally all the feature are ready. However, I got a problem, what if a client had paid before the payment reminder scheduled date? currently, the reminder will still be sent. That's not a great user experience. Thankfully, courier has an automation cancellation feature.

Let's add `cancelAutomationWorkflow` function in `src/lib/courier.ts`.
```ts
export const cancelAutomationWorkflow = async ({
  cancelation_token,
}: {
  cancelation_token: string;
}) => {
    const { runId } = await courierClient.automations.invokeAdHocAutomation({
      automation: {
        // define a cancel action, that send a cancelation_token
        steps: [{ action: 'cancel', cancelation_token }],
      },
    });

   return runId;
};
``` 
What is a cancelation_token? It's a unique token that can be set to an automtation workflow so it cancelable by sending `cancel` action with matching `cancelation_token`. 

Add cancelation_token to `scheduleReminder`, I use invoice's Id as token.
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
I will call `cancelAutomationWorkflow` when an invoice's status is updated to `PAID` in `updateStatus` trpc endpoint. 
```ts
// src/server/trpc/router/invoice.ts

 updateStatus: protectedProcedure
    .input(..) // omitted for brevity
    .mutation(async ({ ctx, input }) => {
      const { invoiceId, status } = input;

      // update an invoice's status in database
      const updatedStatus = await ctx.prisma.invoice.update({
        where: { id: input.invoiceId },
        data: { status: input.status },
        where: { id: invoiceId },
        data: { status },
      });

      // cancel payment reminder automation workflow
      if (status === 'PAID') {

        //call the cancel workflow to cancel the payment remminder for matching cancelation_token.
        await cancelAutomationWorkflow({
          cancelation_token: `${invoiceId}-reminder`,
        });
      }

      return updatedStatus;
    }),
```

It should work.

[SHOW THE CANCELATION LOG]

### Part 5: Error Handling
An important thing to note when doing AJAX request is there are possibilities of failed request/error. I want to handle the error by throwing it to client, so it can be reflected in UI.

On error, Courier API throws error with `CourierHttpClientError` type by default. I will also have the return value in `src/lib/courier.ts` consistent to format below.
```ts
// On Success
type SuccessResponse = { data: any, error: null }

// On Error
type ErrorResponse = { data: any, error: string }
``` 
Now, I can add this `try-catch` block to all function in `src/lib/courier.ts`.
```ts
try {
  // ..function code
  
  // modified return example
  return { data: runId, error: null };

} catch (error) {
  // make sure it's an error fron Courier
  if (error instanceof CourierHttpClientError) {
      return { data: error.data, error: error.message };
    } else {
      return { data: null, error: "Something went wrong!" };
    }
}
```
Let's see a handling example on `sendEmail` trpc endpoint.
```ts
// src/server/trpc/router/invoice.ts

  const { error: sendError } = await sendInvoice(..);
  if (sendError) throw new TRPCClientError(sendError);

  const { error: scheduleError } = await scheduleReminder(..);
  if (scheduleError) throw new TRPCClientError(scheduleError);
```

Interested with the project? contribute here [https://github.com/fazzaamiarso/invoys](https://github.com/fazzaamiarso/invoys)






## About the Author

My name is Fazza Razaq Amiarso. I'm a full stack developer. I'm also an Open Source contributor currently maintaining [invoys](https://github.com/fazzaamiarso/invoys) and [astro-reactive-library](https://github.com/ayoayco/astro-reactive-library). 

I love to share my knowledge and learning [in my blog](https://fazzaamiarso.me/). Also, I occasionally helping other developers in [FrontendMentor](https://www.frontendmentor.io/profile/fazzaamiarso) in my free-time.

## Quick Links

🔗 [github repo](https://github.com/fazzaamiarso/invoys)

🔗 [project motivation](https://fazzaamiarso.me/projects/invoys)