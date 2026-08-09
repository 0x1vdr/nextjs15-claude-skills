# nextjs15-claude-skills

> System-level instruction rulesets and `.claude/skills` context layer engineered for Next.js 15 (App Router), React 19, and Server Actions.

[![Vercel Release](https://img.shields.io/badge/Vercel-v15.0.0-black?logo=vercel)](https://vercel.com)
[![Claude Compatibility](https://img.shields.io/badge/Claude-3.5%20Sonnet%20%2F%20Opus-purple)](https://anthropic.com)
[![React 19 Ready](https://img.shields.io/badge/React-19.0.0-61dafb?logo=react)](https://react.dev)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## Overview

`nextjs15-claude-skills` provides zero-hallucination context protocols for Claude 3.5 Sonnet and Cursor. It forces LLMs to adhere strictly to Vercel's production standards, eliminating outdated Next.js 13/14 patterns (such as synchronous `cookies()`, legacy `pages/` routing, or improper RSC boundaries).

Once linked, your AI client automatically injects strict static analysis rules for state management, edge caching, server-side mutations, and parallel routing.

---

## Architecture & Enforced Conventions

### 1. Next.js 15 Async Request APIs
- Enforces `await` on all dynamic request APIs (`cookies()`, `headers()`, `params`, and `searchParams`).
- Automates migration from synchronous parameter access to dynamic Promise handling.

### 2. React 19 & Server Action Rules
- Strict separation of data fetching (RSC) and mutations (`'use server'`).
- Native integration of `useActionState`, `useFormStatus`, and optimistic UI updates via `useOptimistic`.
- Automatic input validation using `zod` schemas inside server boundary functions.

### 3. Advanced Caching (`dynamicIO` & `staleTimes`)
- Enforces explicit `unstable_cache` tagging and granular `revalidateTag()` strategies.
- Eliminates silent cache bugs by enforcing default opt-in dynamic behavior where required.

---

## Skill Architecture
.claude/
└── skills/
├── next15-async-params.json       # Rules for async dynamic APIs
├── rsc-boundary-validator.json   # Enforces RSC vs Client Component boundaries
├── server-actions-zod.json       # Type-safe mutations & state handling
└── edge-cache-invalidation.json  # Granular tagging & staleTime policies


---

## Quick Setup

Install via CLI or drop the context pack into your workspace root:
```bash
npx vercel-skills@latest init --framework=nextjs15
Or manually link to your global Claude environment:
mkdir -p ~/.claude/skills
cp -r .claude/skills/* ~/.claude/skills/
// Skill Auto-Activated: [server-actions-zod]
'use server';

import { z } from 'zod';
import { revalidateTag } from 'next/cache';
import { auth } from '@/lib/auth';

const UpdateUserSchema = z.object({
  displayName: z.string().min(2).max(50),
});

export async function updateUserState(prevState: any, formData: FormData) {
  const session = await auth();
  if (!session?.user) throw new Error('Unauthorized');

  const validated = UpdateUserSchema.safeParse({
    displayName: formData.get('displayName'),
  });

  if (!validated.success) {
    return { errors: validated.error.flatten().fieldErrors };
  }

  await db.user.update({
    where: { id: session.user.id },
    data: validated.data,

  });

  revalidateTag(`user-${session.user.id}`);
  return { success: true };
}
Benchmark & Workflow Comparison
Refactoring API Route to Type-Safe Server Action
Prompt: "Convert app/api/user/update/route.ts to a Next.js 15 Server Action with optimistic updates."
// Skill Auto-Activated: [server-actions-zod]
'use server';

import { z } from 'zod';
import { revalidateTag } from 'next/cache';
import { auth } from '@/lib/auth';

const UpdateUserSchema = z.object({
  displayName: z.string().min(2).max(50),
});

export async function updateUserState(prevState: any, formData: FormData) {
  const session = await auth();
  if (!session?.user) throw new Error('Unauthorized');

  const validated = UpdateUserSchema.safeParse({
    displayName: formData.get('displayName'),
  });

  if (!validated.success) {
    return { errors: validated.error.flatten().fieldErrors };
  }

  await db.user.update({
    where: { id: session.user.id },
    data: validated.data,
  });

  revalidateTag(`user-${session.user.id}`);
  return { success: true };
}
Performance MetricsTask Execution PatternWithout Skill PackWith nextjs15-claude-skillsAsync Params RefactoringHigh Hallucination (Uses Sync)100% Valid Next.js 15 SyntaxRSC Boundary SafetyLeaks Client Hooks to ServerStrict RSC/Client IsolationMutation ValidationMissing Zod / Manual ParsingAutomated Type-Safe SchemasCold-Start Latency SetupGeneric / Non-EdgeEdge-Optimized RulesetsContributingMaintained for the Next.js developer community. Pull requests for new skill rulesets or edge runtime extensions are welcome.
