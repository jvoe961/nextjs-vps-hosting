# next js hosting: Vercel vs a flat-rate Vultr VPS, with real costs, SSR trade-offs and setup steps

Most "best Next.js hosting" lists hand you the same six names and stop there. What they rarely explain is the decision that actually shapes your bill: managed platforms meter requests, bandwidth, build minutes and team seats, while an always-on Node.js server on a VPS charges one flat number every month. Your app runs the same either way. Your invoice won't.

This guide walks through both routes with current prices, shows what the metered hosts really cost once a team and real traffic show up, and covers the self-hosting path on Vultr in detail: plans, trade-offs, and the exact deployment steps. If you want the shortcut first:

> **Quick picks**
>
> - Personal project or MVP, low traffic: Vercel's Hobby tier is free (100 GB transfer, non-commercial use) and takes minutes to set up. Hard to argue with.
> - Growing app, more than one developer, or a budget that shouldn't swing month to month: run Next.js on a Vultr VPS. The High Frequency 1 GB plan costs **$6/mo**, billed hourly and capped, with no renewal-price jump. It handles most small-to-mid SSR apps comfortably.
> - 👉 [Compare Vultr's current Cloud Compute plans and prices](https://www.vultr.com/pricing/?ref=7268283)

## What Next.js actually needs from a host

Short version: any web server can host a static export of a Next.js site, but only a host with a real Node.js runtime can host the full framework. Before comparing providers, check what your app actually uses, because the requirements differ a lot:

- **Node.js 18.17 or newer** — Next.js 15 won't run without it; Node 20 LTS is the recommended choice for production.
- **A persistent Node process** — SSR, Server Components, Server Actions and API routes all need a server that stays up between requests. Static-only hosts can't fake this.
- **Build headroom** — `next build` on a medium project wants at least 1 GB of RAM, sometimes more. This is the requirement cheap plans fail first.
- **A place to cache ISR pages** — Incremental Static Regeneration needs a persistent file system or a KV store. Anything ephemeral breaks it.
- **Edge runtime support** — Next.js Middleware and Edge API routes only run properly at the edge on Vercel, Cloudflare and Netlify. VPS setups use ordinary Node middleware instead.

Static export (`output: 'export'`) deserves its own sentence: it deploys anywhere, including basic shared hosting, but you lose SSR, API routes, ISR, Middleware and Image Optimization. Fine for docs and landing pages. Wrong for anything with a login or per-request data.

## What the managed platforms really cost

Vercel is free until it isn't: the Hobby tier covers personal projects well, and costs start stacking through seats and usage once the app becomes a business.

| Plan | Price | What's included | The catch |
| --- | --- | --- | --- |
| Hobby | $0 | 100 GB transfer, 1M function invocations, 5,000 image optimizations/mo, 10s function timeout | Personal, non-commercial use only |
| Pro | $20/user/mo + usage | 1 TB bandwidth, 10M edge requests, $20 usage credit, 60s timeouts, preview deploys | Every deploying seat costs another $20/mo; bandwidth overage from $0.15/GB |
| Enterprise | Custom | Custom limits, compliance, support | Sales process; you already know if you're in this tier |

The seat pricing is what surprises teams. One developer on Pro is $20/mo. Add a second deploying teammate and you're at $40 before a single gigabyte of overage; a five-person team starts around $100/mo before any usage. Vercel also reworked its underlying compute pricing in early 2026 (Fluid Compute), so function costs now depend on how efficiently your code waits on I/O — better than before, but still a meter running.

The alternatives, briefly:

- **Cloudflare Workers** runs Next.js through the OpenNext adapter. The paid plan starts at $5/mo, static asset requests are free and unlimited, and there's no per-GB egress billing, which makes it the strongest value at high traffic. The trade-offs: you're on an adapter rather than the native platform, and workers cap at 3–10 MB in size.
- **Netlify** charges $9/mo Personal or $20/mo Pro flat with unlimited members, so seats don't stack. Overage works out to roughly $0.13/GB.
- **Railway** is a container platform with a similar workflow: $5/mo Hobby including $5 of usage, and typical Next.js apps land around $8–15/mo. Egress runs $0.05/GB, and there's no built-in CDN edge layer.
- **AWS Amplify** is the one to check carefully: it officially supports Next.js 12 through 15 only, with streaming and on-demand ISR on its documented unsupported list. If you're on Next.js 16, look elsewhere.

One warning about "cheap" hosts in general: some budget shared hosts advertise teaser rates that multiply at renewal. FastComet, for example, lists a Starter plan at $1.79/mo that renews at $8.95 — a 400% jump hiding in the fine print. Always check the renewal column before comparing headline prices.

Vercel does hold one legitimate, hard-to-copy advantage: the company maintains Next.js itself, and it's the only platform listed as a verified adapter in Next.js's own deployment docs. New framework features work there on day one with zero configuration. If your app leans on the newest framework behavior, that fidelity is worth real money.

## Self-hosting Next.js on a VPS: what changes

You trade per-request billing for a fixed server bill, and you take on the ops work the managed platforms do for you. That's the whole deal in one sentence. Here's what it means in practice.

On an always-on Node.js server, the full Next.js runtime just works: SSR and streaming responses, Server Actions, API routes, ISR against the local file cache, WebSockets for dashboards and chat, and `next/image` optimization running through sharp on your own machine with no per-image line item on anyone's invoice. There's no function timeout to design around either — a long-running export job is just a process on your server.

What you take on:

- OS security updates and Node version upgrades.
- No pull-request preview environments out of the box. Managed platforms generate a preview URL per PR; self-hosted, you build that with extra tooling or live without it.
- ISR caching lives on that one server. A single VPS gives you a single-region cache, which is fine for most apps but isn't a global edge network.
- Uptime is your responsibility. The host keeps the machine up; keeping the app up is on you.

Self-hosting fits apps with steady traffic, teams currently paying per-seat, real-time features, or anyone who wants costs that don't move when traffic spikes. If you ship rarely, hate touching servers, or rely on PR previews daily, a managed platform will still serve you better.

## Vultr plans for running Next.js

Vultr is a cloud infrastructure provider with 32 data center locations worldwide, Linux instances billed by the hour and capped at a fixed monthly rate, and no long-term contracts. You pick a plan, deploy an OS image, and get root on a clean server — which is exactly what full Next.js hosting needs.

The catalog also includes High Performance and dedicated-vCPU Optimized compute lines, plus Bare Metal and Cloud GPU. Those matter for other workloads. For a Next.js app, the two shared-vCPU Cloud Compute lines below are the relevant ones:

| Plan | vCPU / RAM | Storage | Monthly transfer | Price/mo | Deploy |
| --- | --- | --- | --- | --- | --- |
| Regular Performance – 0.5 GB | 1 / 0.5 GB | 10 GB SSD | 0.5 TB | $2.50* | [Check price](https://www.vultr.com/pricing/?ref=7268283) |
| Regular Performance – 0.5 GB | 1 / 0.5 GB | 10 GB SSD | 0.5 TB | $3.50 | [Check price](https://www.vultr.com/pricing/?ref=7268283) |
| Regular Performance – 1 GB | 1 / 1 GB | 25 GB SSD | 1 TB | $5 | [Check price](https://www.vultr.com/pricing/?ref=7268283) |
| Regular Performance – 2 GB | 1 / 2 GB | 55 GB SSD | 2 TB | $10 | [Check price](https://www.vultr.com/pricing/?ref=7268283) |
| Regular Performance – 4 GB | 2 / 4 GB | 80 GB SSD | 3 TB | $20 | [Check price](https://www.vultr.com/pricing/?ref=7268283) |
| Regular Performance – 8 GB | 2 / 8 GB | 160 GB SSD | 4 TB | $40 | [Check price](https://www.vultr.com/pricing/?ref=7268283) |
| Regular Performance – 16 GB | 4 / 16 GB | 320 GB SSD | 5 TB | $80 | [Check price](https://www.vultr.com/pricing/?ref=7268283) |
| High Frequency – 1 GB | 1 / 1 GB | 25 GB NVMe | 1 TB | $6 | [Check price](https://www.vultr.com/pricing/?ref=7268283) |
| High Frequency – 2 GB | 2 / 2 GB | 50 GB NVMe | 2 TB | $12 | [Check price](https://www.vultr.com/pricing/?ref=7268283) |
| High Frequency – 4 GB | 2 / 4 GB | 75 GB NVMe | 3 TB | $24 | [Check price](https://www.vultr.com/pricing/?ref=7268283) |
| High Frequency – 8 GB | 4 / 8 GB | 150 GB NVMe | 4 TB | $48 | [Check price](https://www.vultr.com/pricing/?ref=7268283) |

*\*The $2.50 tier is IPv6-only. The $3.50 version adds IPv4, which you'll want if your visitors are on regular home and mobile networks.*

Notes worth reading before you pick:

- Pricing is hourly, capped at the monthly rate. Deploy a $6 instance for an afternoon of testing, pay a few cents, destroy it. There's no annual commitment and no promo-to-renewal price switch.
- Automatic backups cost an extra 20% of the instance price — $1.20/mo on the $6 plan. Cheap insurance; turn it on.
- Regular Performance uses Intel cores with standard SSDs. High Frequency uses faster cores with NVMe storage, and for Node workloads it's the better default at nearly the same price.

Which plan for which app:

- Static export or a small SSR site (blog, docs, marketing pages): the $5–6 tiers are plenty.
- Typical SSR app with a few dynamic routes and a database on the same box: $12–24. The 2 GB of RAM matters more than the extra CPU here.
- Anything heavier — Redis, multiple containers, a busy Postgres — starts at $24–48, or keep the database on Vultr's separate managed database product and let the app server stay small.

That build-memory point is the one people miss. A 1 GB instance runs a small Next.js app all day, but a medium-sized `next build` can exhaust 1 GB mid-compile. Either add swap or build in CI and ship the standalone output. 👉 [Open Vultr's Cloud Compute lineup](https://www.vultr.com/products/cloud-compute/?ref=7268283) to see the full spec sheet for each tier.

## What a year of Next.js hosting actually costs

Put the realistic numbers side by side and the picture gets clearer:

| Platform | Entry production price | How it meters | Fine print worth knowing |
| --- | --- | --- | --- |
| Vultr VPS (self-host) | $6/mo (High Frequency, 1 GB) | Hourly, capped at the monthly rate | You run updates; auto-backups add 20% |
| Vercel Pro | $20/user/mo + usage | Seats, bandwidth past 1 TB, function usage | Five developers ≈ $100/mo before any usage |
| Cloudflare Workers | $5/mo minimum | Requests + CPU time; no per-GB egress | Runs via the OpenNext adapter; 3–10 MB worker size cap |
| Netlify | $9 Personal / $20 Pro | ~$0.13/GB past included credits | Pro is flat with unlimited members |
| Railway | $5/mo (incl. $5 usage) | CPU/RAM usage; $0.05/GB egress | Typical Next.js app lands around $8–15/mo |
| AWS Amplify | Usage-based, no base fee | $0.15/GB served, build minutes | Officially supports Next.js 12–15; no streaming, no on-demand ISR |

A solo developer self-hosting on the $6 High Frequency plan spends $72/year, or $86 with backups enabled. Vercel Pro solo is $240/year before overages. The gap widens with people: three deploying developers on Vercel cost $60/mo — $720/year before usage — while Netlify Pro stays flat at $240, and a Vultr box doesn't care how many people deploy to it.

The honest counterweight: self-hosting costs time. Budget an evening for the initial setup and a little ongoing attention for updates. 👉 [Check Vultr's live pricing](https://www.vultr.com/pricing/?ref=7268283) and confirm the current rates for the region you have in mind.

## How to deploy Next.js on a Vultr instance

The whole flow takes an hour or less once you've done it once:

1. Deploy an Ubuntu 24.04 LTS instance in the region closest to your users. The $6 High Frequency plan is the usual starting point.
2. Install Node 20 LTS — NodeSource's setup script or nvm both work.
3. Clone your repo and run `npm ci && npm run build`. If the build runs out of memory on a 1 GB plan, add 2 GB of swap or build in CI and deploy the standalone output folder.
4. Run the app under PM2 or a systemd unit (`next start -p 3000`) so it comes back after crashes and reboots.
5. Put Caddy in front as a reverse proxy — it handles HTTPS certificates automatically. Nginx plus certbot works too, if you prefer.
6. Point your domain's A record at the instance IP, and use Vultr's firewall to expose only ports 22, 80 and 443.
7. Optional but recommended: enable automatic backups and take a snapshot before every framework upgrade.

Testing on a throwaway instance costs pennies per hour, which makes the whole decision low-stakes. 👉 [Create your Vultr account and deploy a test instance](https://bit.ly/VulTR), and you'll have real numbers instead of guesses.

## FAQ

### Can a Vultr instance run full Next.js SSR?

Yes. It's a plain Linux server with Node on it, so everything the framework can do locally — SSR, streaming, Server Actions, API routes, ISR, image optimization — runs the same. No platform layer sits in between removing features.

### Does Vultr have a free tier for Next.js hosting?

No permanent free tier exists. Hourly billing is the practical substitute: a $6/mo plan used for one day of testing costs roughly 20 cents, and you can destroy the instance whenever you want.

### Do ISR and next/image still work on a VPS?

Yes, both work. ISR caches to the server's local file system, and image optimization runs through sharp on your own machine with no per-image fee. The one limitation: the ISR cache lives on that single instance, so a rebuild or migration starts the cache fresh.

### Should a beginner start on Vercel or Vultr?

Vercel's Hobby tier, unless you specifically want to learn server administration. It's free for personal projects and removes the entire ops layer. Move to self-hosting when the app gains real traffic, a team, or a budget line that needs to be predictable.

## The short version, by situation

- Personal site, portfolio, or MVP with light traffic: stay on the free managed tiers. Paying for hosting before you need to is wasted money.
- A business app that leans on brand-new Next.js features: a managed platform earns its premium — just model the seat costs before inviting teammates.
- Real traffic, multiple developers, WebSockets or long-running jobs, or a budget that must not swing: put the app on a Vultr VPS. The $6–12 High Frequency plans cover most Next.js deployments, and the price you see is the price you keep paying.

Two things to check before you order: whether your build needs more than 1 GB of RAM (if unsure, take the 2 GB plan), and which region your users actually live in. Then 👉 [pick a plan and get your server online](https://www.vultr.com/pricing/?ref=7268283).
