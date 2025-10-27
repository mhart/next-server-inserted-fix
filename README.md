# A fix for Next.js inserted html streaming

Based on [the Next.js benchmark from t3dotgg](https://github.com/t3dotgg/cf-vs-vercel-bench/tree/main/next-bench) (with the bulk of the "math" CPU stuff removed, leaving mostly just rendering work).

This illustrates the impact of fixing [the logic when rendering the inserted html stream](https://github.com/vercel/next.js/blob/498349c375e2602f526f64e8366992066cfa872c/packages/next/src/server/app-render/make-get-server-inserted-html.tsx#L80-L98).

[A PR has been created against Next.js](https://github.com/vercel/next.js/pull/85394) with (essentially) this fix.

## To run before the fix

```bash
pnpm install
pnpm build
NODE_ENV=production node .next/standalone/server.js
```

## Testing the fix

```bash
patch -p0 < diff.patch
NODE_ENV=production node .next/standalone/server.js
```

(run the patch command after building – run it again if you want to reverse the patch)

## Results

Default wrk settings (2 threads, 10 connections)

```bash
wrk -d30 http://localhost:3000
```

- Avg before patch: 1.94s
- Avg after patch: 1.65s (1.18x faster)

With `node --single-threaded` to simulate a single core (like on Lambda) and then running a single request at a time (1 thread, 1 connection):

```bash
wrk -c1 -t1 -d30 http://localhost:3000
```

- Avg Before: 215ms
- Avg After: 183ms (1.17x faster)

### Other platforms

Running a few benchmarks, I see:

- Deno: 1.38x faster
- workerd: 1.24x faster
- Bun: 1.15x faster
