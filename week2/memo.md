Hello Product Team,

Given our current budget to use two A10G GPUs to generate and update a 512 x 512 background within one second, dynamic to the user's prompt, I propose we use a pretrained SDXL Turbo model as our primary approach, as the standard 1000-step DDPM would fail our current budget.

The main reason the standard DDPM fails is due to the sequential generation steps. This is the main bottleneck that would lead DDPM in standard form to fail, further proven below assuming ~300 ms for typing detecting:

(1000 steps * 0.05 s for typing) + 0.3 s for display/processing = ~50 seconds >> 1 second req.

Each denoising updated would depend on previous outputs, so even using the full two separate GPUs in budget would not get rid of the sequential step bottleneck, as each update depends on all previous steps. Thus, each denoising step requires the result of the earlier step (like a Markov chain) so this presents a huge bottleneck in our budget needs.

I propose we deploy a pretrained SDXL Turbo, which would use diffusion distillation to support generation in significantly fewer steps. The Turbo model does this by combining diffusion modeling with an adversarial training. Given our budget and timeline, I expect that with SDXL Turbo, our total latency would be ~0.6 seconds (assuming denoising takes 300 ms, but now taking 1 step to update: 1 step * 0.3 s + overhead = 0.6). 

I estimate that it would take about one business week (~72 engineering hours) for the majority of implementation and deployment of the model, where week two and three can be focused on benchmarking and ensuring the model meets the rest of our requirements. 

Thus, these estimates leave a significantly larger latency margin for use to ship. We would want to benchmark each project and measure FID, visual quality, and how well the model responds to prompting against held-out backgrounds that represent our training distribution, but I believe the Turbo would be these best while we evaluate quality in latter weeks. We would want to keep a close eye on the FID to ensure that we meet the <30 threshold as a part of the budget.

I propose that these changes are focused on week 5 and week 6, just so that the key risks (highlighted below) behind how the UI would work with the model could work.

I think the key risk here would be a possible background missing a user-prompt while mainitnaing realism, but I think this could be something we include as an "undo" button in the UI. The idea here would be to keep the previous background such that if the user's prompt is not influencing the current image in the prototype, the user can return to the previous image generated from the last prompt. 

As a fallback, I also propose we could use a ten-step DDIM, withich would apply DDIM sampling to a pretrained model. The main difference here to DDPM would be a change to the sampling procedure to move from 1,000 tiny steps to fewer and larger updates (not a single one like Turbo but still much fewer than 1,000, providing us a latency margin in budget). The reason I keep this as a fallback is the epxected computation for quality trade-off that can be observed with DDIM, while Turbo provides a different architecture to DDPM that could provide quality (which is why I propose as our primary). 

