# Pricing & Credits

## Pricing & Credits – ChainGPT AI NFT Generator

ChainGPT’s AI NFT Generator (API & SDK) uses a **credit-based pricing model** for all image generation and utilities. This means every request consumes **ChainGPT Credits (CGPTc)**, where **1 credit = $0.01 USD**. The pricing is **consistent across the API, SDK, and web app** – there is no markup for API usage. Developers only pay per use (pay-as-you-go), making costs transparent and easy to track.

{% hint style="info" %}
_Note: Basic free daily usage is available on the platform for single-image generation, but programmatic use beyond those limits will consume credits. If you want to try out our models & application for free, visit :_ [_https://NFT.ChainGPT.org_](https://nft.chaingpt.org)_._&#x20;
{% endhint %}

***

### Image Generation Pricing

ChainGPT’s NFT Generator offers **three AI models** for image creation, each suited to different needs:

* **VeloGen** – a fast, efficient model for quick drafts.
* **NebulaForge XL** – a high-fidelity model (XL) for premium, detailed outputs.
* **VisionaryForge** – a creative model for imaginative, high-quality visuals.

Despite their differences in speed and quality, **all in-house models cost the same base amount in credits per image**. You can customize each generation by adjusting the **number of diffusion steps** (which refines quality) and the output **resolution**, without changing the base credit cost. The table below summarizes pricing per image for each model, including optional enhancements:

<table><thead><tr><th width="127.3515625">Model</th><th width="86.2421875">Steps</th><th width="282.01953125">Resolution</th><th>Credits Per Image</th></tr></thead><tbody><tr><td><strong>VeloGen</strong></td><td>1–2</td><td>512×512, 512×768, 640×640</td><td>0.4 Credits</td></tr><tr><td><strong>VeloGen</strong></td><td>3–4</td><td>512×512, 512×768, 640×640</td><td>0.6 Credits</td></tr><tr><td><strong>VeloGen</strong></td><td>1–2</td><td>1024×1024, 1024×768</td><td>0.8 Credits</td></tr><tr><td><strong>VeloGen</strong></td><td>3–4</td><td>1024×1024, 1024×768</td><td>1 Credit</td></tr><tr><td><strong>NebulaForge XL</strong></td><td>1–25</td><td>1024×1024, 768×1024, 1024×768</td><td>1 Credit</td></tr><tr><td><strong>NebulaForge XL</strong></td><td>26–50</td><td>1024×1024, 768×1024, 1024×768</td><td>1.25 Credits</td></tr><tr><td><strong>VisionaryForge</strong></td><td>1–25</td><td>1024×1024, 768×1024, 1024×768</td><td>1 Credit</td></tr><tr><td><strong>VisionaryForge</strong></td><td>26–50</td><td>1024×1024, 768×1024, 1024×768</td><td>1.25 Credits</td></tr></tbody></table>

{% hint style="info" %}
💡 **Tip:** You can significantly enhance image quality or double the resolution by applying the **Upscale (1x or 2x)** utility feature—see details below.
{% endhint %}

**Key Points:**

* **Base Generation Cost:** Generating an image with any of the three models costs **1 credit per image** by default. This covers any prompt and any supported base resolution, regardless of how many steps are used. (Using more steps yields higher quality but may take longer; it does _not_ increase the credit cost.)
* **Upscaling Cost:** If you request **HD/Enhanced output**, the system will automatically apply the upscaler. This roughly doubles the cost – **+1 credit for a single enhancement** (so 2 credits total). All models support upscaling; for instance, VeloGen can upscale a 512px image to \~1920px. Upscaling twice (for even larger images) adds another credit (total 3 credits), though one pass is usually sufficient.
* **Model Selection:** Using **VeloGen** is most **cost-efficient** for rapid prototyping or bulk previews – it generates images extremely fast with fewer diffusion steps. **NebulaForge XL** and **VisionaryForge**, on the other hand, produce more **premium-quality** images with finer detail (supporting up to 50 steps of refinement). They are ideal for final renders or when image fidelity is paramount. **All three models cost 1 credit per base image**, so you can choose the model based on your quality/speed needs without worrying about price differences. _(Third-party models (e.g. OpenAI’s DALL·E) are also available for compatibility, but they have a separate pricing structure and are not the focus here.)_

***

### Utilities Pricing

In addition to image generation, the ChainGPT NFT API provides several utility features to enhance your results or enforce content guidelines. Below is a breakdown of these tools, including which are free and which consume credits:

<table><thead><tr><th width="131.6328125">Utility Tool</th><th width="128.375">Credit Cost</th><th>Description &#x26; Usage</th></tr></thead><tbody><tr><td><strong>Prompt Enhancement</strong></td><td>+<strong>0.5 credits</strong> per prompt</td><td><em><strong>Enhance Prompt</strong></em><strong> –</strong> Uses an AI language model to refine or expand your text prompt before image generation. This optional step can make prompts more descriptive or creative. It costs a half-credit per use.</td></tr><tr><td><strong>1x Image Upscaling</strong></td><td>+<strong>1 credit</strong> per upscale</td><td><em><strong>HD Enhancement</strong></em><strong> –</strong> Improves image resolution and quality using a super-resolution model (R-ESRGAN). If requested as part of generation, this is counted in the image’s credit cost (as detailed above).</td></tr><tr><td><strong>2x Image Upscaling</strong></td><td>+<strong>2</strong> <strong>credits</strong> per upscale</td><td><em><strong>HD Enhancement</strong></em><strong> –</strong> Improves image resolution and quality using a super-resolution model (R-ESRGAN). If requested as part of generation, this is counted in the image’s credit cost (as detailed above).</td></tr><tr><td><strong>NSFW Detection</strong></td><td><strong>Free</strong> (0 credits)</td><td><em><strong>Content Filter</strong></em><strong> –</strong> Every image generation is automatically screened for NSFW or disallowed content using a Vision-Transformer classifier. This ensures that <strong>NSFW materials are blocked</strong> and not generated. The NSFW filter does not consume credits. (However, disallowed prompts or outputs will be filtered out, so ensure your prompts comply with content guidelines to avoid wasted requests.)</td></tr></tbody></table>

{% hint style="info" %}
**Note:** _The prompt enhancer and upscaler are optional tools to help you get better results with fewer tries. They incur small credit fees as noted. The NSFW safety filter is built-in for compliance and does not charge credits._
{% endhint %}

***

### Model Comparison

All three models share the same base credit cost (1 credit per image). Select a model based on your project’s balance of speed and quality.

<table><thead><tr><th width="125.40625">Model</th><th>Best For</th><th>Pros ✅</th><th>Cons ⚠️</th></tr></thead><tbody><tr><td><strong>VeloGen</strong></td><td>Good quality images, Rapid prototyping, quick previews, high-volume generation</td><td>- Fastest generation- Cost-efficient for bulk tasks- Good results even at lower steps</td><td>- Limited detail compared to other models- Best results at lower resolutions (512–1024px)</td></tr><tr><td><strong>NebulaForge XL</strong></td><td>Premium-quality NFT images, detailed artwork, final production</td><td>- Highest image detail &#x26; fidelity- Supports large resolutions (up to 1536px with upscale)- Ideal for production-level quality</td><td>- Slower than VeloGen (especially at high step counts)- Higher steps (26–50) slightly increase cost (+0.25 credits)</td></tr><tr><td><strong>VisionaryForge</strong></td><td>Premium-quality NFT images, artistic visuals, final production</td><td>- Similar premium quality to NebulaForge XL- Excellent fine detail and color accuracy- Supports high-resolution outputs (up to 1536px)</td><td>- Slower generation times similar to NebulaForge XL- Higher steps (26–50) slightly increase cost (+0.25 credits)</td></tr></tbody></table>

***

### Credit Purchase Options

ChainGPT Credits (CGPTc) can be purchased directly through the ChainGPT dashboard. **Credits never expire**, and you can buy them on a one-time or recurring basis to suit your integration’s needs. Key points about purchasing credits:

* **Credit Bundles:** Credits are available in flexible amounts. For example, **1,000 credits cost $10 USD** (since 1 credit = $0.01) – you can scale up as needed (e.g. 10,000 credits = $100) or purchase smaller/larger packs in the app. There is no minimum usage fee; you only pay for the credits you need, and you can top-up anytime. _(For enterprise or high-volume needs, larger bulk packages are available.)_
* **Payment Methods:** You can pay via **credit/debit card** or **cryptocurrency**. Supported crypto includes stablecoins like USDT/USDC and popular coins (e.g. ETH, BNB, TRX). Additionally, you may purchase credits using **$CGPT (ChainGPT’s native token)**. All purchases are processed through the ChainGPT Crypto AI Hub interface for convenience.
* **Discounts for $CGPT or Subscriptions:** If you choose to pay with **$CGPT tokens** _or_ if you enable an **automatic monthly credit top-up** subscription, you receive a **15% credit bonus/discount** on those purchases. This means your money goes further (for example, a $100 purchase via $CGPT yields 11,500 credits instead of 10,000). This incentive is designed to reward ecosystem users and those who plan for recurring usage.
* **Monthly Plans:** For organizations or power-users, you can set up a **monthly credits plan** (auto-purchase). This ensures your account is refilled every month with a set number of credits, so your integration never runs out. The 15% discount applies here as well. _(Alternatively, ChainGPT’s **Freemium** membership – achieved by staking CGPT tokens – provides a monthly allotment of credits, see Membership docs for details.)_

All credit transactions and balances can be managed via your ChainGPT account dashboard. Credits are deducted in real-time as you make API/SDK calls, and you can monitor usage in the dashboard to plan when to top-up.

***

### Best Practices & Tips

Follow these recommendations to efficiently use ChainGPT’s NFT Generator and maximize your credits:

**1. Optimize Prompts with Enhancement:** Before generating images repeatedly, use the **Prompt Enhancement** tool (1 credit). It helps create detailed, effective prompts on your first try, reducing unnecessary spending.

**2. Prototype Quickly with VeloGen:** Start initial tests and bulk generations with **VeloGen** (1 credit per image). It's fast and cost-effective, allowing quick iteration. Once satisfied, switch to **NebulaForge XL** or **VisionaryForge** for high-quality final outputs.

**3. Use Image Upscaling Strategically:** Only use upscaling (enhancement) when higher resolution or improved clarity is essential. For drafts or previews, skip enhancement to save credits. Apply **1× Upscale** for improved quality or **2× Upscale** for exceptional detail and resolution.

**4. Adhere to Content Guidelines:** Ensure prompts follow content guidelines to avoid blocked generations. Non-compliant prompts may still consume credits without producing results. Use clear, appropriate, family-friendly prompts.

**5. Track Usage Regularly:** Monitor your credit usage via the ChainGPT dashboard. Regular tracking helps you manage credits effectively and anticipate future needs. For consistent usage, set up automatic monthly top-ups to benefit from a 15% discount.

**6. Validate with Samples Before Bulk Generation:** When creating large NFT collections, always generate a small sample batch first. Validate prompt effectiveness and style consistency before committing credits to large-scale generation, ensuring your final batch meets your expectations.

{% hint style="info" %}
By applying these best practices, you’ll achieve optimal results and predictable costs while making the most of ChainGPT’s powerful NFT generation tools.
{% endhint %}
