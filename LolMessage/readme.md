# Build an Open Source NGL.link alternative with Next.js and the Courier API

## Background

For [Courier Hack](https://courier-hacks.devpost.com/)s, I decided to build an open source `NGL.link` alternative, which was fun to build, easy to use, and free for everyone.

[Contribute here >](https://github.com/n4ze3m/lol/)

`NGL.link` allows users to create an inbox for anonymous questions. Every user receives a link to their inbox, which can be shared across social media platforms, but lately, people have been using it for Instagram specifically.

Similar to `NGL.link`, Lol Message allows users to create an inbox for anonymous questions. Every user receives a link to the inbox, which can be shared across social media platforms. Users can also view all the received messages in a dashboard with email notifications. Lol Message reminds users about new messages via email.

Here's a quick demo of the project:

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/dPTf3qporVE/0.jpg)](https://www.youtube.com/watch?v=dPTf3qporVE)

I decided to use the following tools and technologies for this project:

- Notifications System: [Courier](https://courier.com/)
- Web Framework: [Next.js](https://nextjs.org/)
- Backend Framework: [tRPC](https://trpc.io/)
- Authentication & Database: [Supabase](https://supabase.io/)
- UI: [Mantine](https://mantine.dev/)

If you are not familiar with these technologies, I’d recommend checking out their documentation before continuing.

## Instructions

We will be building the core features of the project in this article. You can find the source code of the full project [here](https://github.com/n4ze3m/lol), and you can find the live demo [here](https://lol-message.vercel.app/).

Let's get started!

The diagram below shows the architecture which we will be building in this article.

![Architecture Diagram](https://i.imgur.com/WtTIASB.png)

When a person submits a message, it will send a request to the tRPC server. The tRPC server will then send a request to the Courier API to send a email notification to the recipient and save the message in the database. The recipient will receive an email notification and can view the message in the dashboard.

The project divided into 4 parts:

1. Project Setup
2. Courier Setup
3. Backend
4. Frontend

### Part 1: Project Setup

In this first part, we will be setting up the project and creating the database. We will be using Supabase for the database and authentication and Next.js for web framework.

- Create a new next.js project using the following command:

```bash
yarn create next-app --typescript lol-mini
```

- *Note* I used yarn, but you can use npm as well or any other package manager.


After creating the Next.js project, we will be setting up the Supabase database.

In this article, I'm not going to cover the Supabase setup and user authentication. You can check out the [documentation](https://supabase.com/docs) for that. We are going to use the `prisma` client to interact with the database.

- Install the `prisma` client using the following command:

```bash
yarn add prisma -D
```


Next, we need to create a new prisma schema file. You can use the following command to create a new prisma schema file:

```bash
yarn prisma init
```

After creating the prisma schema file, we need to create two models. The first model will be for the `user` and the second model will be for the `message`.



Open the `schema.prisma` file and add the following code for the `user` model


```prisma
model user {
  id           String    @id @default(dbgenerated("uuid_generate_v4()")) @db.Uuid
  username     String
  email        String
  email_notify Boolean   @default(true)
  question     String    @default("Send me anonymous messages lol!")
  pause        Boolean   @default(false)
}
```



The `user` model has the following fields:
- `id`: The unique id of the user (primary key) randomly generated using `uuid_generate_v4()`
- `username`: The username of the user
- `email`: The email of the user
- `email_notify`: A boolean field to enable or disable email notifications
- `question`: The question that will be displayed on the user's inbox
- `pause`: A boolean field to pause the inbox

Next, we need to add the `message` model to the `schema.prisma` file.

```prisma
model message {
  id         String   @id @default(dbgenerated("uuid_generate_v4()")) @db.Uuid
  message    String
  opened     Boolean  @default(false)
  user_id    String
  question   String   @default("Send me anonymous messages lol!")
  user       user  @relation(fields: [user_id], references: [id])
  created_at DateTime @default(now()) @db.Timestamptz(6)
}
```

The `message` model has the following fields:
- `id`: The unique id of the message (primary key) randomly generated using `uuid_generate_v4()`
- `message`: The message that the user sent
- `opened`: A boolean field to check if the message is opened or not
- `user_id`: The id of the user who received the message
- `question`: The question that will be displayed on the user's inbox
- `user`: The relation between the `message` and `user` models
- `created_at`: The date and time when the message was created

Since we have a relation between the `message` and `user` models, we need to update the `user` model to include the `messages` field.

```diff
model user {
  id           String    @id @default(dbgenerated("uuid_generate_v4()")) @db.Uuid
  username     String
  email        String
  email_notify Boolean   @default(true)
  question     String    @default("Send me anonymous messages lol!")
  pause        Boolean   @default(false)
+ message     message[]
}
```

The `message` field is an array of `message` objects that belong to the user. 


Final code of the `schema.prisma` file should look like this:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model user {
  id           String    @id @default(dbgenerated("uuid_generate_v4()")) @db.Uuid
  username     String
  email        String
  email_notify Boolean   @default(true)
  question     String    @default("Send me anonymous messages lol!")
  pause        Boolean   @default(false)
  message      message[]
}

model message {
  id         String   @id @default(dbgenerated("uuid_generate_v4()")) @db.Uuid
  message    String
  opened     Boolean  @default(false)
  user_id    String
  question   String   @default("Send me anonymous messages lol!")
  user       user  @relation(fields: [user_id], references: [id])
  created_at DateTime @default(now()) @db.Timestamptz(6)
}
```

Next, we need to migrate the database. 

Make sure you have working database credentials in the `.env` file. You can find the database credentials in the Supabase dashboard. For more information, check out the [documentation](https://supabase.com/docs/guides/integrations/prisma).

Now, we can migrate the database using the following command:


```bash
yarn prisma migrate dev --name init
```

Your Prisma schema is now in sync with your database schema and you can start querying your database with Prisma Client.


- *Note* You need to add a user in order to use the application because we are not going to cover the user authentication in this article. You can use prisma studio or supabase dashboard to add a user. 

- *Note* To use supabase auth with next.js, you can check out supabase auth helper details [here](https://github.com/supabase/auth-helpers)


Ah! We are done with the project setup. Let's move on to the next part.



### Part 2: Courier Setup

In this part, we will be setting up the Courier account and  integrating  with Gmail API to send email notifications.

- Log in to your Courier account and create a workspace.
- For onboarding, select the email channel and let Courier build with Node.js. Start with the Gmail API since it only takes seconds to set up. All we need to do to authorize is log in via Gmail. Now the API is ready to send messages.
- Copy the starter code, a basic API call using cURL, and paste it in a new terminal. It already has saved your API key and knows which email address you want to send to, and has a message already built in.

Once you can see the dancing pigeon, you are ready to use Courier.


After setting up the Courier account, we will be creating a new notification template in the Courier dashboard.

- Go to the [Courier dashboard](https://app.courier.com/) and click on the `Create Notification` button.

- Select the `Email` channel and click on the `publish changes` button.

- After publishing the changes, click on the email icon from the left sidebar to create a new email template.

Your email template should look like this without logo:

![Template](https://i.imgur.com/RJBR4aQ.png)

Now we will be adding content to the email template.You can use the following format:

![Template 1](https://i.imgur.com/DbhGtt7.png)

![Template 2](https://i.imgur.com/TlZaIbx.png)


Small explanation of the above template:

- `{username}` is the username of the user who received the message.

- `{question}` is the question that the user asked.

- `{subject}` is the subject of the message randomly generated.

-  `{url}` is the link to the message page.

Lastly, we need to copy the `template ID` and `Authorization` token from the Courier dashboard.

For that click on the `Send` tab, and you will see the `template ID` and `Authorization` token.

![Send Tab](https://i.imgur.com/HCLeWvn.png)

Copy the template ID and Authorization token and paste it in `.env` file.

Use the following format:

```
COURIER_API_KEY=YOUR_AUTH_TOKEN
COURIER_TEMPLATE_ID=YOUR_TEMPLATE_ID
```

Ok now we are done with the Courier setup. Let's move on to the next part.

### Part 3: Backend

This part is the most important part of the project. We will be building the backend API using [tRPC](https://trpc.io/) and integrating it with [Courier](https://courier.com/) to send email notifications.

*Note*: You can use any backend framework of your choice


So let's get started with the backend API.

First, we need to install the `trpc` and configure it with our Next.js project.

Please check out the following documentation to configure `trpc` with Next.js [here](https://trpc.io/docs/v9/nextjs).


After successfully configured `trpc` with Next.js, we need to install `Courier` client using the following command:

```bash
yarn add @trycourier/courier
```

Ok now we are ready to build the backend API.

First, we need to create a new file `utils/database.ts` and add the following code:

```ts
import { PrismaClient } from "@prisma/client";

declare global {
  var prisma: PrismaClient | undefined;
}

export const database =
  global.prisma ||
  new PrismaClient({
    log: ["query"],
  });

if (process.env.NODE_ENV !== "production") global.prisma = database;
```

The above code is used to connect to the database using `prisma` client. P


- *Note*:  When we migrate the database for the first time, prisma will install the `@prisma/client` package. If you are getting an error, please install the `@prisma/client` package using the following command:

```bash
yarn add @prisma/client
```

Next open your trpc router file and it should look like this:


```ts
import * as trpc from "@trpc/server";
import * as trpcNext from "@trpc/server/adapters/next";
import { CourierClient } from "@trycourier/courier";
import { database } from "utils/database";
import { z } from "zod";

export const appRouter = trpc
  .router()
// export type definition of API
export type AppRouter = typeof appRouter;

// export API handler
export default trpcNext.createNextApiHandler({
  router: appRouter,
  createContext: () => null,
});
```

We need to add a query and mutation to the router file. So let's do that.

#### findUserByUsername

This query is used to find the user by username. It takes the username as an input and returns the user object.
  
In your router file, add the following code:

```ts
.query("findUserByUsername", {
    input: z
      .object({
        username: z.string().nullish(),
      })
      .nullish(),
    resolve: async ({ input }) => {
      if (!input?.username) {
        return null;
      }
      const user = await database.profile.findFirst({
        select: {
          id: true,
          username: true,
          question: true,
          pause: true,
        },
        where: {
          username: input.username,
        },
      });

      if (!user) {
        throw new trpc.TRPCError({
          code: "NOT_FOUND",
          message: "User not found",
        });
      }

      return user;
    },
  })
```


#### answerQuestion

This mutation is used to answer the question. It takes the `userId`, `question`, and `answer` as an input and returns the message object.

If the user has enabled the email notification, then it will send the email notification using the `Courier` client.

So let's add the following code to the router file:

```ts
.mutation("answerQuestion", {
    input: z
      .object({
        userId: z.string().nullish(),
        question: z.string().nullish(),
        answer: z.string().nullish(),
      })
      .nullish(),
    resolve: async ({ input }) => {
      if (!input?.userId || !input?.question || !input?.answer) {
        return null;
      }

      const user = await database.profile.findFirst({
        where: {
          id: input.userId,
        },
      });

      if (!user) {
        throw new trpc.TRPCError({
          code: "NOT_FOUND",
          message: "User not found",
        });
      }

      const answer = await database.message.create({
        data: {
          message: input.answer,
          user_id: input.userId,
          question: input.question,
        },
      });

      if (user.email_notify) {
        const courier = CourierClient({
          authorizationToken: process.env.COURIER_API_KEY,
        });

        const url = process.env.NEXT_PUBLIC_HOSTNAME
          ? `https://${process.env.NEXT_PUBLIC_HOSTNAME}/me/a`
          : "http://localhost:3000/me/a";

        await courier.send({
          message: {
            to: {
              email: user.email,
            },
            template: process.env.COURIER_TEMPLATE_ID as string,
            data: {
              subject: "You have a new message",
              username: user.username,
              url: `${url}/${answer.id}`,
              question: input.question,
            },
          },
        });
      }

      return {
        message: "Message sent successfully",
      };
    },
  });
```


After implementing both the queries and mutations, your router file should look like this:

```ts
import * as trpc from "@trpc/server";
import * as trpcNext from "@trpc/server/adapters/next";
import { CourierClient } from "@trycourier/courier";
import { database } from "utils/database";
import { z } from "zod";

export const appRouter = trpc
  .router()
  .query("findUserByUsername", {
    input: z
      .object({
        username: z.string().nullish(),
      })
      .nullish(),
    resolve: async ({ input }) => {
      if (!input?.username) {
        return null;
      }
      const user = await database.profile.findFirst({
        select: {
          id: true,
          username: true,
          question: true,
          pause: true,
        },
        where: {
          username: input.username,
        },
      });

      if (!user) {
        throw new trpc.TRPCError({
          code: "NOT_FOUND",
          message: "User not found",
        });
      }

      return user;
    },
  }).mutation("answerQuestion", {
    input: z
      .object({
        userId: z.string().nullish(),
        question: z.string().nullish(),
        answer: z.string().nullish(),
      })
      .nullish(),
    resolve: async ({ input }) => {
      if (!input?.userId || !input?.question || !input?.answer) {
        return null;
      }

      const user = await database.profile.findFirst({
        where: {
          id: input.userId,
        },
      });

      if (!user) {
        throw new trpc.TRPCError({
          code: "NOT_FOUND",
          message: "User not found",
        });
      }

      const answer = await database.message.create({
        data: {
          message: input.answer,
          user_id: input.userId,
          question: input.question,
        },
      });

      if (user.email_notify) {
        const courier = CourierClient({
          authorizationToken: process.env.COURIER_API_KEY,
        });

        const url = process.env.NEXT_PUBLIC_HOSTNAME
          ? `https://${process.env.NEXT_PUBLIC_HOSTNAME}/me/a`
          : "http://localhost:3000/me/a";

        await courier.send({
          message: {
            to: {
              email: user.email,
            },
            template: process.env.COURIER_TEMPLATE_ID as string,
            data: {
              subject: "You have a new message",
              username: user.username,
              url: `${url}/${answer.id}`,
              question: input.question,
            },
          },
        });
      }

      return {
        message: "Message sent successfully",
      };
    },
  });
// export type definition of API
export type AppRouter = typeof appRouter;

// export API handler
export default trpcNext.createNextApiHandler({
  router: appRouter,
  createContext: () => null,
});
```

And that's it! We have successfully implemented the API for our application. Now let's move on to the Frontend.

### Part 4: Frontend

Finally, we are done with the backend API. Now it's time to build the frontend. 


So for the frontend, we are going to use [Mantine](https://mantine.dev/).

Check out the following documentation to setup Mantine and Mantine Form with Next.js [here](https://mantine.dev/guides/next/).


Ok now we are ready to build the frontend.


let's create a new file `pages/u/[username].tsx` and add the following code:

```tsx
import { UserQuestionProfileBody } from "components/UserQuestionProfileBody";
import Head from "next/head";
import { GetServerSideProps, NextPage } from "next/types";
import React from "react";
import { database } from "utils/database";


export const getServerSideProps: GetServerSideProps = async (context) => {
  const { username } = context.query;
  const user = await database.profile.findFirst({
    where: {
      username: username as string,
    },
  });
  if (!user) {
    return {
      redirect: {
        destination: "/",
        permanent: false,
      },
    };
  }
  return {
    props: {},
  };
};

const UserQuestionProfile: NextPage = () => {
  return (
    <div>
      <Head>
        <title>Hmmm</title>
      </Head>
      <UserQuestionProfileBody />
    </div>
  );
};

export default UserQuestionProfile;
```

The above code is used to check if the user exists or not. If the user exists, then it will render the `UserQuestionProfileBody` component otherwise it will redirect to the home page.


Next, let's create a new file `components/UserQuestionProfileBody.tsx` and add the following code:

```tsx
import {
  Card,
  Container,
  Skeleton,
  Text,
  Button,
  Textarea,
  ActionIcon,
  Group,
  Avatar,
} from "@mantine/core";
import { useForm } from "@mantine/form";
import { useRouter } from "next/router";
import React from "react";
import { trpc } from "utils/trpc";

export const UserQuestionProfileBody = () => {
  const router = useRouter();

  const { data: userProfile } = trpc.useQuery(
    [
      "getProfileByUsername",
      {
        username: router.query.username as string,
      },
    ],
    {
      enabled: Boolean(router.query.username),
    }
  );

  const { mutateAsync: createMessage, isLoading: isSubmittingAnswer } =
    trpc.useMutation("answerQuestion", {
      onSuccess: () => {
        form.reset();
      },
    });

  const form = useForm({
    initialValues: {
      answer: "",
    },
    validate: {
      answer: (value) => {
        if (value.length === 0) {
          return "Answer is required";
        }
      },
    },
  });

  return (
    <React.Fragment>
      <Container size={500} mt={80}>
        <form
          onSubmit={form.onSubmit(async (values) => {
            await createMessage({
              answer: values.answer,
              question: userProfile?.question,
              userId: userProfile?.id,
            });
          })}
        >
          <Skeleton visible={!userProfile}>
            <Card withBorder shadow="sm" radius="md">
              <Card.Section withBorder inheritPadding py="md">
                <Group>
                  <Avatar
                    radius="xl"
                    src={`https://avatars.dicebear.com/api/jdenticon/${userProfile?.id}.svg?background=%230000ff`}
                  />
                  <div>
                    <Text size="sm" color="dimmed">
                      {userProfile?.username}
                    </Text>
                    <Text>{userProfile?.question}</Text>
                  </div>
                </Group>
              </Card.Section>
              <Card.Section inheritPadding py="md">
                <Textarea
                  autosize
                  minRows={2}
                  placeholder="What's on your mind?"
                  {...form.getInputProps("answer")}
                />
              </Card.Section>
            </Card>
          </Skeleton>
          <Skeleton visible={!userProfile} my="md">
            <Button
              type="submit"
              loading={isSubmittingAnswer}
              color="teal"
              fullWidth
              disabled={userProfile?.pause}
            >
              Send
            </Button>
            {userProfile?.pause && (
              <Text align="center" size="sm" color="red" my="sm">
                Oh no! This user has paused their inbox.
              </Text>
            )}
            <Text my="md" align="center" size="sm" color="dimmed">
              {"100% anonymous q&a"}
            </Text>
          </Skeleton>
        </form>
      </Container>
    </React.Fragment>
  );
};

```


Ok, let's understand what the above code does.

- We are using the `trpc` client to fetch the user profile by username.

- We are using the `Mantine Form` to create the form and handle the form submission.

- We are using the `trpc` client to create the message.

- We are using the `Mantine` components to build the UI.


And that's it. We have successfully built the lol message mini app.

If everything goes well, you should be able to see the following UI:

![Final UI](https://i.imgur.com/RIx79WL.png)

After submitting the message, email notification will be sent to the user and it will look like this:

![Email Notification](https://i.imgur.com/WANmJEB.png)

## Conclusions

In this tutorial, we have learned how to build a simple anonymous question and answer app with email notification using Next.js, tRPC and Courier.

So that's it for this tutorial. I hope you enjoyed it. 

## About the Author



My name is [Nazeem](https://n4ze3m.site) and I am a Software Engineer. I am passionate about building web apps and I love to share my knowledge with others. You can find me on [Twitter](https://twitter.com/n4ze3m) and [GitHub](https://github.com/n4ze3m).

## Quick Links

- [Source Code](https://github.com/n4ze3m/lol)

- Next.js [Documentation](https://nextjs.org/docs)

- tRPC [Documentation](https://trpc.io/docs/v9/nextjs)

- Mantine [Documentation](https://mantine.dev/)

- Courier [Documentation](https://www.courier.com/docs)

- Supabase [Documentation](https://supabase.io/docs)

- Supabase Auth [Documentation](https://github.com/supabase/auth-helpers)

- Prisma Studio [Documentation](https://www.prisma.io/docs/concepts/components/prisma-studio)

- Lol Message Devpost [Link](https://devpost.com/software/lol-message)
