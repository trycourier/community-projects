# Lol Message


## Background

During the [Courier Hack](https://courier-hacks.devpost.com/), I wanted to build a project that was fun and easy to use. So I decided to build an open source `NGL.link` alternative with email notification and a dashboard to view all the received messages. 

I decided to use the following tools and technologies for this project:

- Notification: [Courier](https://courier.com/)
- Web Framework: [Next.js](https://nextjs.org/)
- Backend Framework: [tRPC](https://trpc.io/)
- Authentication & Database: [Supabase](https://supabase.io/)
- UI: [Mantine](https://mantine.dev/)

## Instructions

We will be building core features of the project in this tutorial. You can find the source code of the full project [here](https://github.com/n4ze3m/lol), and you can find the live demo [here](https://lol-message.vercel.app/).

So let's get started!

Below diagram shows the architecture which we will be building in this article.

![Architecture Diagram](https://i.imgur.com/WtTIASB.png)


The project divided into 4 parts:

1. Courier Setup
2. Project Setup
3. Backend
4. Frontend 


### Part 1: Courier Setup

In this first part, we will be setting up the Courier account and  integrating  with Gmail API to send email notifications.

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

### Part 2: Project Setup

In this part, we will be setting up the project and creating the database. We will be using Supabase for the database and authentication and Next.js for web framework.

- Create a new next.js project using the following command:

```bash
yarn create next-app --typescript lol-mini
```

*Note* I used yarn, but you can use npm as well or any other package manager.


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

After creating the prisma schema file, we need to add the following code to the `schema.prisma` file:

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

To to successfully migrate the database, we need to add the `DATABASE_URL` in the `.env` file.  You can check out the following documentation about how to intergate the `prisma`  with Supabase [here](https://supabase.com/docs/guides/integrations/prisma).

After adding the `DATABASE_URL` in the `.env` file, we need to migrate the database using the following command:

```bash
yarn prisma migrate dev --name init
```


*Note* You need to add a user in order to use the application because we are not going to cover the user authentication in this article. You can use prisma studio or supabase dashboard to add a user.

Ah! We are done with the project setup. Let's move on to the next part.


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

The above code is used to connect to the database using `prisma` client.

Next open your trpc router file and add the following code:

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
  })
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

// export type definition of API
export type AppRouter = typeof appRouter;

// export API handler
export default trpcNext.createNextApiHandler({
  router: appRouter,
  createContext: () => null,
});
```

We have added one query and one mutation in the above code. Let's understand what each of them does.

#### findUserByUsername

This query is used to find the user by username. It takes the username as an input and returns the user object.

#### answerQuestion

This mutation is used to answer the question. It takes the `userId`, `question`, and `answer` as an input and returns the message object.

If the user has enabled the email notification, then it will send the email notification using the `Courier` client.


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


<!--  create a about link -->

My name is [Nazeem](https://n4ze3m.site) and I am a software engineer who loves to build things.

## Quick Links

- [Source Code](https://github.com/n4ze3m/lol)

- Next.js [Documentation](https://nextjs.org/docs)

- tRPC [Documentation](https://trpc.io/docs/v9/nextjs)

- Mantine [Documentation](https://mantine.dev/)

- Courier [Documentation](https://www.courier.com/docs)

- Supabase [Documentation](https://supabase.io/docs)

- Supabase Auth [Documentation](https://github.com/supabase/auth-helpers)
