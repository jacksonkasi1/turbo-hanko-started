
---

# app > api > trpc > route.ts

```ts
import { appRouter, createContext } from "@repo/server";
import { createNextApiHandler } from "@trpc/server/adapters/next";

// export API handler
export default createNextApiHandler({
  router: appRouter,
  createContext,
});
```

---

# layout.ts

```ts
import type { Metadata } from 'next'
import "@/styles/globals.css";

import TRPCProvider from "./provider";

export const metadata: Metadata = {
  title: 'Create Next App',
  description: 'Generated by create next app',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div>
      <TRPCProvider>
        {children}
      </TRPCProvider>
    </div>
  )
}

```


---

# page.tsx

```ts
"use client";

import Link from "next/link";

// import { CreateTodo } from "@/components/create-post";
import Todos from "@/components/Todos";


export default async function Home() {

  return (
    <main className="flex min-h-screen flex-col items-center justify-center bg-gradient-to-b from-[#2e026d] to-[#15162c] text-white">
      <div className="container flex flex-col items-center justify-center gap-12 px-4 py-16 ">
        <h1 className="text-5xl font-extrabold tracking-tight sm:text-[5rem]">
          Create <span className="text-[hsl(280,100%,70%)]">T3</span> App
        </h1>
        <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 md:gap-8">
          <Link
            className="flex max-w-xs flex-col gap-4 rounded-xl bg-white/10 p-4 hover:bg-white/20"
            href="https://create.t3.gg/en/usage/first-steps"
            target="_blank"
          >
            <h3 className="text-2xl font-bold">First Steps →</h3>
            <div className="text-lg">
              Just the basics - Everything you need to know to set up your
              database and authentication.
            </div>
          </Link>
          <Link
            className="flex max-w-xs flex-col gap-4 rounded-xl bg-white/10 p-4 hover:bg-white/20"
            href="https://create.t3.gg/en/introduction"
            target="_blank"
          >
            <h3 className="text-2xl font-bold">Documentation →</h3>
            <div className="text-lg">
              Learn more about Create T3 App, the libraries it uses, and how to
              deploy it.
            </div>
          </Link>
        </div>
        <CrudShowcase />
      </div>
    </main>
  );
}

async function CrudShowcase() {
  return (
    <div className="w-full max-w-xs">
      <Todos />
      <br />
      <hr />
      <br />
      {/* <CreateTodo /> */}
    </div>
  );
}

```


---

# components > Todos.tsx

```ts
"use client";

import React from "react";
import { trpc } from "@/utils/trpc";

import Todo from "./Todo";

const Todos = () => {


  const { data: todoList = [], isLoading, error } = trpc.todo.all.useQuery();


  if (isLoading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>Error: {error.message}</p>;
  }

  return (
    <div>
      {todoList.length ? (
        todoList.map((todo) => {
          return <Todo key={todo.id} todo={todo} />;
        })
      ) : (
        <p>Create a todo.</p>
      )}
    </div>
  );
};

export default Todos;

```

----

# utils.ts

```ts
import { createTRPCReact } from '@trpc/react-query'
import type { AppRouter } from '@repo/server'

export const trpc = createTRPCReact<AppRouter>()
```

---

# server > src > router > todo.ts

```ts
import { router, publicProcedure, protectedProcedure } from "../trpc";
import { z } from "zod";
import { todoInputSchema } from "../types";

export const todoRouter = router({
  all: publicProcedure.query(async ({ ctx }) => {
    const todos = await ctx.prisma.todo.findMany({});
    return [{
        id: 1,
        text: "Todo 1",
        done: false,
      }
    ];
  }),
  ...code
```
---

# server > src > router > index.ts

```ts
import { router } from "../trpc";
import { postRouter } from "./post";
import { authRouter } from "./auth";
import { todoRouter } from "./todo";

export const appRouter = router({
  post: postRouter,
  auth: authRouter,
  todo: todoRouter,
});

// export type definition of API
export type AppRouter = typeof appRouter;

```


# server > src > context.ts

```ts
import { prisma } from "@repo/db";
import { type inferAsyncReturnType } from "@trpc/server";
import { type CreateNextContextOptions } from "@trpc/server/adapters/next";

/**
 * Replace this with an object if you want to pass things to createContextInner
 */
type AuthContextProps = {
  auth: "";
};

/** Use this helper for:
 *  - testing, where we dont have to Mock Next.js' req/res
 *  - trpc's `createSSGHelpers` where we don't have req/res
 * @see https://beta.create.t3.gg/en/usage/trpc#-servertrpccontextts
 */
export const createContextInner = async ({ auth }: AuthContextProps) => {
  return {
    auth,
    prisma,
  };
};

/**
 * This is the actual context you'll use in your router
 * @link https://trpc.io/docs/context
 **/
// eslint-disable-next-line @typescript-eslint/no-unused-vars
export const createContext = async (opts: CreateNextContextOptions) => {
  return await createContextInner({ auth: "" }); // Remove getAuth
};

export type Context = inferAsyncReturnType<typeof createContext>;
```

----

# server > src > trpc.ts

```ts
import { initTRPC, TRPCError } from "@trpc/server";
import { type Context } from "./context";
import superjson from "superjson";

const t = initTRPC.context<Context>().create({
  transformer: superjson,
  errorFormatter({ shape }) {
    return shape;
  },
});

const isAuthed = t.middleware(({ next, ctx }) => {
  // todo: need to implement auth
  // if (!ctx.auth) {
  //   throw new TRPCError({ code: "UNAUTHORIZED", message: "Not authenticated" });
  // }
  return next({
    ctx: {
      auth: ctx.auth,
    },
  });
});

export const router = t.router;
export const publicProcedure = t.procedure;
export const protectedProcedure = t.procedure.use(isAuthed);
```

# server >  index.ts

```ts
export type { AppRouter } from "./src/router";
export { appRouter } from "./src/router";

export { createContext } from "./src/context";
export type { Context } from "./src/context";
```

i in Todos.ts i am keep getting loading only. i can't get data from trpc. how to give. give me code. tell the error. give me fixed version of code