# Vultr vs RunPod: Same-Card Pricing, Egress and Idle Costs, and Which GPU Workload Belongs Where (Vultr Plan Table Included)

On the one GPU both providers rent, RunPod is cheaper: an L40S 48GB costs $1.09/hr on RunPod's Secure Cloud and $0.79/hr on its Community Cloud, against $1.671/hr on Vultr Cloud GPU. Vultr's on-demand GPU line starts at that same L40S, then $1.71/hr for an A40 48GB and $2.40/hr for an A100 80GB PCIe. RunPod's cheapest listed rate is $0.34/hr for an RTX 4090 on Community Cloud.

Hourly price favors RunPod on nearly every card the two share. What changes the math is everything around the card: how much data leaves the machine, whether the GPU bills while it sits idle, and whether the rest of your stack already lives in the same account. Below are the verified rates on both sides, the three costs the pricing pages don't put next to each other, and the workload profiles that point in different directions.

> **Quick pick:** if you run a persistent GPU service and already host compute, storage or Kubernetes on Vultr, the L40S 48GB at $1.671/hr is the default. It is slightly cheaper per hour than the older A40 ($1.71/hr) and comes with 16 vCPU and 180 GB RAM. Step up to the A100 80GB PCIe at $2.40/hr only when 48 GB of VRAM won't hold your model.
> 👉 [Check current Vultr Cloud GPU prices by card](https://www.vultr.com/pricing/?ref=9738262-9J)

## The full Vultr Cloud GPU lineup

Vultr's Cloud GPU documentation names four cards: A16, A40, A100 and L40S. Three of them sell at a flat per-GPU hourly rate that stays the same whether you take one card or eight.

| Plan | GPU memory | vCPU · RAM | Price | Order |
| --- | --- | --- | --- | --- |
| L40S (1, 2, 4 or 8 GPUs) | 48 GB | 16 vCPU · 180 GB | **$1.671/GPU/hr** | [L40S 48GB plan](https://www.vultr.com/products/cloud-gpu/nvidia-l40s?ref=9738262-9J) |
| A100 80GB PCIe (1 to 8 GPUs) | 80 GB | 12 vCPU · 120 GB | **$2.40/GPU/hr** | [A100 pricing and specs](https://www.vultr.com/pricing/?ref=9738262-9J) |
| A40 (1 or 4 GPUs) | 48 GB | 24 vCPU · 120 GB | **$1.71/GPU/hr** | [A40 pricing and specs](https://www.vultr.com/pricing/?ref=9738262-9J) |

Each instance also carries local storage and an outbound allowance: 1.20 TB and 10 TB/month on the L40S, 1.40 TB and 10 TB on the A100 PCIe, 1.40 TB and 15 TB on the A40. Multi-GPU totals follow the same per-card rate, so an 8-card A100 PCIe node runs $19.178/hr and four A40s run $6.849/hr.

The A16 is the outlier. It's rented as a fractional vGPU slice, which is how Vultr lets a small job pay for part of a physical card instead of all of it. Slice rates appear in the deploy flow rather than in a published fixed table, so budget from the console figure you actually see.

Linux instances are reached over SSH, Windows instances over Remote Desktop, and GPU images ship with drivers preinstalled. Vultr's GPU pricing page also lists two 8-GPU HGX nodes, H100 80GB SXM and B200 192GB, at $2.30 and $3.50 per GPU-hour. Those figures carry a condition. Both come from a block where "starting from" means prepaid contract terms, and Vultr's public plans API prices the same H100 node at $23.92/hr ($2.99 per GPU) with on-demand deployment switched off, and the B200 node at $68.00/hr ($8.50 per GPU). Read the page number as a floor negotiated against a commitment, and the API number as what a self-serve account may actually be charged.

## Same card, two invoices

| GPU | Vultr | RunPod Secure Cloud | RunPod Community Cloud |
| --- | --- | --- | --- |
| L40S 48GB | $1.671/hr | $1.09/hr | $0.79/hr |
| A100 80GB PCIe | $2.40/hr | $1.79/hr | $1.19/hr |
| H100 80GB SXM | from $2.30/GPU/hr on contract; $2.99/GPU self-serve per Vultr's API | $3.99/hr | $2.69/hr |
| B200 (192 GB on Vultr, 180 GB on RunPod) | from $3.50/GPU/hr on contract; $8.50/GPU self-serve per Vultr's API | $7.99/hr | $5.98/hr |

Vultr figures were listed in October 2026 and re-checked in November 2026; RunPod figures come from its own pricing page in October 2026. RunPod sells capacity from two pools: Secure Cloud in vetted data centres, and Community Cloud on third-party hosts, which is cheaper and less predictable.

The L40S row is the cleanest comparison because both sides rent the same 48 GB card with no contract qualifier. RunPod is about 35% cheaper on Secure Cloud and less than half the price on Community. The A100 gap is wider still, $2.40 against $1.19, roughly 50%.

H100 and B200 need the footnote because Vultr's page price and its API price differ by a factor of three to five. RunPod's $3.99/hr Secure Cloud H100 lands between Vultr's contract floor and its self-serve rate, which is why "Vultr is cheaper on H100" is only true if you sign a prepaid commitment.

## Three costs that sit outside the hourly rate

**Egress.** RunPod charges nothing for ingress or egress. Vultr gives each GPU instance an outbound allowance (10 TB/month on L40S and A100, 15 TB on A40) and bills $10 for every TB past it. Pull weights once and that allowance is plenty; stream rendered frames or serve multi-gigabyte model responses to users and it disappears, with 20 TB out on a 10 TB plan adding $100 to the month.

**Idle time.** A Vultr GPU is a virtual machine. It bills for as long as it exists, and nothing winds it down when requests stop arriving. RunPod's Serverless endpoints bill per second while workers run and scale to zero when traffic stops. If your GPU is genuinely busy 20% of the month, Vultr charges you for the other 80%. For a bursty internal tool or a low-traffic API, that gap dwarfs the per-hour difference between the two providers.

**The services around the GPU.** Closest-match monthly list prices:

| Service | RunPod | Vultr |
| --- | --- | --- |
| Small VM | $43.20/mo | $5.00/mo |
| Medium VM | $86.40/mo | $40.00/mo |
| Large VM | $172.80/mo | $160.00/mo |
| Block storage | $10.00/mo | $10.00/mo |
| Object storage | not offered | $18.00/mo |
| Managed PostgreSQL | not offered | $60.00/mo |
| Managed Redis | not offered | $15.00/mo |
| Load balancer | not offered | $10.00/mo |

These are third-party approximations and shift with region and configuration. The pattern holds: a small or medium app server beside the GPU is radically cheaper on Vultr, and the database, cache and load balancer arrive on the same invoice with no cross-provider networking to set up.

## What Vultr Cloud GPU doesn't offer

Documented variants stop at the A100 generation. There is no H200, no B300 and no consumer RTX card in the Cloud GPU line, so anyone hunting the cheapest possible hour for image generation or a 141 GB card for a large model won't find it here.

Multi-node work goes through a different product. Vultr Clusters puts L40S or bare metal nodes on low-latency networking, and it requires a reservation or a special request to deploy. InfiniBand isn't documented for Cloud GPU instances, and neither is a managed training scheduler.

Nothing scales to zero. Every GPU is a VM you start and stop yourself.

RunPod covers all three: H200 at $5.29/hr Secure, B300 at $8.99/hr, instant clusters up to 64 GPUs, and handler containers that bill per second. If your workload is spiky inference behind an API, or a training job that needs more than eight GPUs in a single job, RunPod is the better fit and the per-hour gap is a secondary concern.

## What users actually rate

G2 samples are lopsided, 228 reviews for Vultr against 26 for RunPod on the "meets requirements" question, so read these as directional. Vultr scores 8.8 on meets requirements to RunPod's 8.5, 9.1 on ease of use to 9.3, and 8.1 on support quality to 7.8. RunPod leads on product direction, 9.1 to 8.1. Vultr's negative tags cluster on customer support (5 mentions) and payment issues (3).

A separate weighted comparison across 11 criteria put RunPod at 75 and Vultr at 69. The biggest single gap was GPU availability, 82 to 58. Vultr scored higher on documentation (78 to 70), maturity (76 to 66) and portability (78 to 70); RunPod scored higher on price transparency (84 to 66). That mix matches what the rate cards show: Vultr is the steadier, better-documented platform with a narrower catalogue.

## Which one to pick

- A model endpoint, rendering queue or Windows desktop that stays warm all day, with the rest of your stack already on Vultr: **L40S 48GB at $1.671/hr**.
- More than 48 GB of VRAM on a single card, one node: **A100 80GB PCIe at $2.40/hr**.
- Container-friendly inference with spiky traffic, where idle time should cost nothing: RunPod Serverless.
- H200, B300, a 64-GPU cluster or a cheap consumer card: not Vultr Cloud GPU.

Two checks before you pay. Confirm the card you want is offered in the location you plan to deploy to, since Vultr runs 30+ locations and the console shows which GPUs are in stock where. And confirm whether an H100 or B200 quote is contract pricing or self-serve, because the page figure and the API figure differ enough to break a budget.

On signup credit: Vultr's new-customer promotional credit has been reported by users and third-party write-ups to exclude GPU instances and to expire within about 30 days. Budget list price for your first GPU instance and treat any credit as a bonus for the non-GPU parts of the stack.

👉 [Start with the L40S 48GB plan at $1.671/hr](https://www.vultr.com/products/cloud-gpu/nvidia-l40s?ref=9738262-9J)

## FAQ

**Can I put several GPUs on one Vultr instance?** Yes. L40S comes in 1, 2, 4 and 8-GPU configurations at the same $1.671 per GPU-hour, A100 PCIe up to 8 cards, A40 up to 4.

**Is Vultr cheaper than RunPod for the same card?** No. On the L40S 48GB, Vultr charges $1.671/hr against RunPod's $1.09/hr Secure Cloud rate; on the A100 80GB PCIe, $2.40/hr against $1.19/hr Community Cloud. Vultr's advantage shows up in the surrounding stack, not the GPU hour.

**Can a GPU instance share storage and networking with my other Vultr servers?** Yes. Block storage, object storage, file systems, private networking and the API all sit in the same account, which is the main reason to keep a GPU next to an existing Vultr deployment.
