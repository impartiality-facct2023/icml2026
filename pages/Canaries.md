## Rebuttal
	- ### Response to AkJH {{renderer :wordcountchar_}}
		- We thank the reviewer for their insightful review, especially the connections made to the poisoning literature. We have some clarifications, as well as some clarifying questions that would help us better understand the reviewer's concerns.
		- **We do not propose a new MIA evaluation or attack.** The canaries that we produce are in the input space, and as a result can be used in any MIA. For evaluation, we adopt Aerni et al. 2024  including code and implementation. Our only contribution to the evaluation is the training loop which is of course necessary.
		- > key details are unclear: reference‑model design, loss modeling, global vs. per‑instance thresholding, member prevalence, and confidence intervals (CIs) at very low FPR. Results also omit difficulty calibration, which materially impacts low‑FPR performance.
			- The shadow models ("reference-model design") are the same architecture except in transferability where the architecture is documented.
			- Since we optimize exactly one instance, "member prevalence" is not well-defined.
- ## Reviews
	- ### Reviewer_AkJH
		- **Overall Recommendation:** 3
		- **Confidence:** 4
		- #### Summary
		    The paper tackles privacy auditing for machine learning by optimizing canaries; training samples inserted to test for training-data leakage, so that they are maximally detectable by membership inference attacks (MIAs). The core idea is to pose canary selection as a bilevel optimization: the inner loop trains a model while the outer loop modifies a candidate canary to maximize a membership-likelihood ratio. Practically, the method combines influence‑function–based pre‑selection of promising candidates with unrolled optimization and memory‑saving techniques.
		- #### Scores
			- Soundness: 2
			- Presentation: 3
			- Significance: 2
			- Originality: 2
		- #### Strengths and Weaknesses
		    Strengths:
			- Uses LiRA and reports low‑FPR metrics, which aligns with current best practices for MIA evaluation and auditing.
			- The problem framing of optimizing canaries to improve low‑FPR detectability aligns with current best practice for evaluating MIAs is clear.
				- ==Michael== We do not evaluate MIAs, we evaluate training procedures.
				- We don't propose a new evaluation scheme, we propose a new input
			- The design is reasonable. The combination of influence‑based pre‑selection with an outer optimization loop is plausible and modular.
			- The author's aim to make canaries that generalize across architectures echoes cross‑architecture effects seen in poisoning literature and, if validated rigorously, could be useful.
			    
			  Weaknesses:
			- Even with LiRA and a DP‑SGD sweep, key details are unclear: reference‑model design, loss modeling, global vs. per‑instance thresholding, member prevalence, and confidence intervals (CIs) at very low FPR. Results also omit difficulty calibration, which materially impacts low‑FPR performance.
				- Member prevelenace: how many memebers vs non-members
					- How likely is a member
					- 50:50 prior is ensured
					- https://claude.ai/share/5e537b81-261e-41a0-9d0d-ff3410cccd70
				- ==TODO== @aernim can you take care of this? ==Michael== Not sure what you mean with 'take care of this'? I have little idea of what the current code is doing.
				- ==Michael== Regarding very low FPR regimes, this is the problem of not doing the 20k shadow models evals anymore; wherever we have that, we can argue that the estimates are reasonably accurate, but with the only 2k or however many shadow models, there's not really anything we can say (other than "it's very expensive")
				- ==Michael== We could indirectly address a lot of those concerns by also reporting global threshold TPRs for the strongest attack everywhere in the appendix. IIRC, those are close to LiRA TPR, which means calibration etc has a much smaller effect.
			- The central "third‑party auditing" claim presumes training‑time canary insertion, which external auditors typically lack; label‑only MIAs provide black‑box alternatives and should be a primary comparison.
				- This is true and we have already discussed this fact.
				- ==TODO== @myaghini  add references to this
				- However it is easier to get access to the full. training prodcuedre. Label only solves a different access problem.
			- Missing ablations on candidate pool size, number of canaries, influence e.g. init vs. random, unroll length, attack budget, and transfer under training‑mismatch (optimizer/augmentation/label‑smoothing). Compute/memory budgets and CI reporting at very low FPRs could be more comprehensive.
				- Number of canaries: First, the choice to optimize a single canary was by design; and in keeping withe the definition of differential privacy. Second, increasing number of canaries can only increase the power of the attacks. But given the demonstrated effectivness (large TPR@0.1FPR values) with only a single optimized sample, increasing the attack power would have made the granularity of the other design ablations (effect of initalization, and different optimization pardigms) harder to detect.
					- influence e.g. init vs. random: This is ablated in Table 4 (in main matter) and more extensively Table 5 (in Aappendix A.4)
					- unroll length: This is already abalated in Figure 11 (in appendix E.8)
					- attack budget: This is language from poisoning. It is unclear what an attack budget means in the privacy auditing context. We kindly ask the reviewer to elaborate their intention.
					- Compute/memeory budgets are reported for each step in Table 6.
					- ==Michael== For number of canaries, we can also argue with the usual "privacy is a worst-case metric", and then continue with "increasing the power" etc. Alternatively, one could also argue that multiple canaries might have undesirable interactions.
		- #### Key Questions for Authors
		    Given the strengths, I have the following concerns and clarifying questions aimed at improving the paper:
			- For black‑box, label‑only access (typical for third‑party audits), how does OptiFluence compare to label‑only MIAs at low FPR?
				- ==TODO== @aernim can we implement a lable-only MIA? Given the optimization step, I think we cannot pursue a gradient method if we did this. So comparison sounds moot? Or are they asking us to  test optimized canaries in a label-only MIA? This seems counter-productive because of course MIA subsumes label-MIA. So I'm not sure I understand.
					- ==Michael== There are different approaches but IMO it strays a bit far from the current setup. For one, I'd already expect the predicted labels to be different for in vs. out with the current optimization procedure. Alternatively, we could e.g. max the hinge score for different labels for in vs. out, etc. And then we could optimize over different e.g. data augmentations etc, which just makes everything more complex (and probably unstable), without really being super interesting. But the main point is that I don't really see why we should care about label-only in our setting in the first place.
					- Label only with some rotation and estimate confidences.
					- The method could be extended to produce rotations but out of scope.
					- It is out of scope for our threat model but it is trivial to modify optifluence to make label only work.
					- ==Michael== In summary, I would not cater to the label-only thing. But there might be something that I'm not aware of.
					- "We only consider a threat model with access to predicted confidences for brevity. However, it is trivial to extend OptiFluence to work in a label-only access model. We are happy to elaborte how if the reviewer is interested."
			- The authors claim cross‑architecture transfer. How robust is transfer when optimizer, data augmentation, or label‑smoothing differ between the canary‑generating setup and the audited model? A cross‑arch × training‑recipe matrix with CIs would help (poisoning literature often observes sensitivity to training mismatches).
				- We thank the reviewer for this question. Indeed the different architectures required different training regimes. WideResNet for example does use label-smoothing, and a different cosine scheduler than the ResNet18 model. Note that this was naturally necessary to achieve acceptable generalization behavior on these models; but importantly, we did not tune the models end-to-end. Such standard model training hyper-paramters were tuned by hand for a single model.
				- ==TODO== test with label smoothing ?!
				- ==Michael== can we also say that this is pretty expensive, b/c we need to train a ton of models for every entry of the matrix? We pick a reasonable set of settings that are in-budget and make the point.
			- What are the GPU hours and peak memory required to optimize K canaries on CIFAR‑10, and what would be needed for ImageNet‑scale? How do unroll length and checkpointing frequency affect performance and compute? Please include a cost‑vs‑gain plot.
				- The requested experiments are already present in Appendix D and E for a mix of results on MNIST and CIFAR10. See Figure 11 and the discussion in E.8 for the effect of the truncation paramter k. Also CIFAR100 results use on CIFAR100 use (by necessity)
					- ==Michael== there seems to be a typo. Also, I think we changed some stuff after submission, so we need to be careful with the word "already"
					- Figure 11 is a proxy otherwise we need to run many many evals. No practical difference.
			- How sensitive are the gains to influence‑based initialization vs. random? To candidate pool size, number of canaries, and attack budget? Please add these ablations to establish which components drive improvements.
				- We explore this ablation in Figure 1 rather clearly. In-Distro produces 2.4%, mislableing increases this sligghtly to 3%, High Influence samples increase this to 19.7%. None of these baselines so far are optimized. With the addition of optimization we are able to reach 99.5%. We even include the In-distro+optimization that achies 5.2%. Table 4 expands on this.
				- Our attack budget is 1 sample by def. see previous.
			- The authors report global‑threshold TPR; what member prevalence did you assume, and how sensitive are results to prevalence and to per‑instance thresholds? Could you also provide 95% CIs at 0.1% (and 0.01%) FPR, and assess sensitivity to prevalence and per‑instance thresholds to ensure conclusions are robust across evaluation protocols?
				- ==TODO== @aernim I am a bit unsure of the terminology used here. Can you tackle this?
				- ==Michael== I am equally unsure lol. I think they mean how many times a canary was member vs. non-member? IIRC it is an equal amount of times; however, this doesn't really matter, b/c we report TRP/FPR and not accuracy (and those values get more accurate somewhat independently with the amount of members/non-members).
			- Can the authors clarify how their outer objective differs from gradient matching and why it yields canaries that are more detectable at low FPR or more transferable. In addition, please include a comparison between gradient‑matching poisoning and influence‑only baselines (no outer optimization).
				- Gradient Matching: $$
				  \min _{\mathcal{S}} \mathbb{E}_\theta\left[D\left(\nabla_\theta \ell(\theta ; \mathcal{T}), \nabla_\theta \ell(\theta ; \mathcal{S})\right)\right]
				  $$
				  
				             The decision variable is a synthetic dataset $\mathcal{S}$; the objective aligns gradient directions between $\mathcal{S}$ and $\mathcal{T}$ across many parameter configurations.
				             
				             Gradient matching asks: *does training on $\mathcal{S}$ produce the same parameter updates as training on $\mathcal{T}$ ?* It optimizes in gradient space directly and does not model what happens to the model after training completes.
				  
				             OptiFluence asks: *does including $(x, y)$ in training produce a detectably different model?* The objective is explicitly the log-likelihood ratio $\Lambda(\theta ; x, y)=p\left(\theta \mid Q_{\text {in }}\right) / p\left(\theta \mid Q_{\text {out }}\right)$, approximated via $\ell_{\text {priv }}$. This means the optimization is over **trained model behavior**, not just gradient alignment.
				  
				             A subtle but important connection: if one approximates $\theta_{D \cup\{x\}}$ via its first-order influence function expansion, $\theta_{D \cup\{x\}} \approx \theta_D-\frac{1}{N} H^{-1} \nabla_\theta L\left(\theta_D ; x\right)$, then $\ell_{\text {priv }}$ becomes:
				  
				             $$
				  \ell_{\text {priv }}(x) \approx \nabla_\theta f\left(\theta_D ; x\right)^{\top} \cdot\left(-\frac{1}{N} H^{-1} \nabla_\theta L\left(\theta_D ; x\right)\right)
				  $$ which is exactly the self-influence $I_f(x ; x)=-\nabla_\theta f^{\top} H^{-1} \nabla_\theta L$ from Eq. 9 of the paper. So the IF-Opt baseline of OptiFluence is literally a first-order approximation of gradient matching's inner product, while the full unrolled OptiFluence goes beyond this by propagating gradients through the entire training trajectory. For a more detailed of this connection, please see Appendix B (espcially Lines 811—846).
				  
				  - #### Limitations
				  Yes  
				  - ### Reviewer_GVKf
				  - **Overall Recommendation:** 5
				  - **Confidence:** 4
				  - #### Summary
				  Canaries in training data are often used to determine if a trained model is memorizing private information; however normally canaries are selected in ad-hoc ways. This paper provides a very nice method for selecting canaries in a principled manner so that their detection at inference time is maximized.  
				  
				  Specifically, this is done by an iterative improvement process where first a model is trained with canaries, and then the canaries are iterated upon to improve a privacy loss; a somewhat surprising result is that the resulting optimized canaries do transfer across architectures.  
				  - #### Scores
				  - Soundness: 3
				  - Presentation: 3
				  - Significance: 4
				  - Originality: 4
				  - #### Strengths and Weaknesses
				  I really like the paper -- the problem is novel and interesting, and highly relevant and the paper produces a plausible method and tests it out on some reasonable (although small datasets). The following are somewhat minor and fixable issues, but overall I think the paper should be accepted.  
				  
				  1. I think the transferability bit is a little bit of an overclaim -- it has only been tested on very very small datasets and classification models, which are nothing like the modern models of today. I do think its okay to say that empirical observation is that the canaries transfer, but I would caution against such a bold claim.
				         - ==Michael== I think there's not really much we can do except acknowledge
				         - If we optimize ca canary on unrelated tasks, transferability breaks down conceptually. In a reasonbly related setting, we scale.
				  2. The membership inference test that is used on the canaries in the experiments section is LiRA. It would be interesting to see how the results change with other MIA tests -- and if they get better or worse.
				         - ==Michael== As before, we could include global threshold, which serves as a natural lower bound for other MIAs. Then we have a "bad" MIA in terms of GT, and a very good MIA in terms of LiRA, and we can argue that results for other MIAs likely don't change very much. Which is also a good argument for the generated canaries (they just work)
				         - Even doing the dumbest MIA (global threshold) we get similar results.  TODO Add global threshold.
				  
				  - #### Key Questions for Authors
				  1. Could you better explain the equation in lines 272-274?  
				  2. I found the logit rescaling step described in lines 188-219 a bit unclear. Could you give some more intuition on why this step is needed / how it affects the results of your experiments?
				  - ==Michael== We made this discussion clearer now. A general rescaling step is needed for TODO(whatever LiRA paper says; reference that). The replacement of max with LSE was not needed and we removed it for clarity.
				  - #### Limitations
				  Yes  
				  - ### Reviewer_uCYZ
				  - **Overall Recommendation:** 5
				  - **Confidence:** 3
				  - #### Summary
				  The paper introduces OptiFluence -- an automated method for generating canaries for membership inference attacks on modern ML systems.  
				  Generating canary attacks is a crucial aspect in the field of privacy, as it can help stress test the privacy guarantees of a training algorithm, and fits into the wider question of whether existing algorithms are more private than their guarantees.  
				  
				  Existing approaches rely on basic techniques such as label flipping for an inlier sample, but the authors show that they can get significantly stronger canaries by utilizing influence functions. In keeping with recent advances in the field of data attribution, the authors use an optimized unrolling-based influence function, yielding a good balance of utility and performance.  
				  
				  Moreover, the authors show that outlier samples generated for one model and training algorithm transfer to different models and training algorithms, allowing for a wider set of use-cases (e.g., optimizing the canary on a smaller/known architecture and applying the canary for larger or unknown architectures).  
				  In particular, they are able to use canaries optimized for a non-private training to test the privacy of models trained with DP-SGD.  
				  - #### Scores
				  - Soundness: 3
				  - Presentation: 3
				  - Significance: 3
				  - Originality: 3
				  - #### Strengths and Weaknesses
				  **Strengths:**  
				  The submission gives a clear contribution to the field of privacy by introducing a practical method for generating adversarial examples in privately trained neural nets. These methods are tested on a wide variety of architectures across several datasets and the authors also show that canaries optimized in one setting transfer well to new settings.  
				  
				  Up to minor comments (listed below), the submission appears sound, well-written, and provides a new approach to a significant problem.  
				  
				  **Weaknesses:**  
				  I think the paper is already strong, but below are a few minor issues.  
				  
				  The biggest issue for me right now is the equation in lines 272-274, which seems incorrect to me:  
				  By definition, we have $$\ell_{priv}(x,y) = f(\theta_{D \cup \{ (x,y) \}}; x, y) - f(\theta_{D}; x, y)$$
					- This is a typo. Thanks.
					  In other words, $\ell_{priv}$ depends on $x$ in two ways: once in the effect it has on the model $\theta$ which is now also trained on $x$ , and a second effect by changing the point at which the trained model is being evaluated.  
					  
					  However, the last step of equation 272-274 applies the chain rule, but only with respect to the dependence of $\theta$ on $x$ , and does not take into account the effect that changing $x$ can have on the score by evaluating at a different point.  
					  
					  If this equation is used as an approximation, that is fine since the main contribution is the fact that the proposed canaries work well, but if that is the case, it should be clarified that this is an approximation. If more tests can be run to see if using the exact gradient improves performance, that would be ideal.  
					  
					  **Minor Comments**:  
					  Line 292 "of over" -> either "of" or "over"  
					  
					  Line 83 in Related Work:  
					  "Carline et al. (2019b) and follow-ups".  
					  In a related works section, it would be good to also cite these follow ups.
		- #### Key Questions for Authors
		  1. Could you better explain the equation in lines 272-274?  
		  2. I found the logit rescaling step described in lines 188-219 a bit unclear. Could you give some more intuition on why this step is needed / how it affects the results of your experiments?
			- ==Michael== We made this discussion clearer now. A general rescaling step is needed for TODO(whatever LiRA paper says; reference that). The replacement of max with LSE was not needed and we removed it for clarity.
		- #### Limitations
		    yes
	- ### Reviewer_PUwg
		- **Overall Recommendation:** 2
		- **Confidence:** 4
		- #### Summary
		    This paper presents a novel principled strategy to optimize privacy canaries used to audit models.  
		    Specifically, the paper formulates privacy canary optimization as a bi-level optimization problem and solves the problem by first selecting a set of candidate canaries using influence functions and subsequently optimizing the candidate canaries using unrolled gradients.  
		    When auditing non-private SGD training, the paper reports a substantial increase in attack success when using their optimized canary compared to in-distribution and naively initialized baselines.
		- #### Scores
			- Soundness: 3
			- Presentation: 3
			- Significance: 1
			- Originality: 2
		- #### Strengths and Weaknesses
		    This is a well written paper and easy to follow despite the relatively complicated methodology used to optimize the canary design.  
		    Furthermore, the experimental results for non-private SGD are very strong, achieving near perfect attack accuracies for MNIST and CIFAR-10.  
		      
		    However, I would say the main weaknesses of the paper are in (1) motivation, (2) breadth of results, and (3) comparison with prior work.  
		      
		    **Motivation**  
		      
		    In my opinion, one of the main gaps in the motivation of the paper lies in convincing the reader *why* they should care about the privacy leakage from potentially meaningless samples.  
		    From what I can tell from Figure 5, the optimized canaries do not really look like anything and I cannot imagine that they can somehow be present in any real-world dataset.  
		    At least for mis-labelled sample baseline, there can be an argument that it can come from annotator error or something so there is some level of real-world impact.  
		    However, I am not sure what will be the real-world impact of showing that you can nearly perfectly identify when a meaningless sample is present or not in a dataset.  
		    I understand the authors mention regulatory auditing as a motivating factor, but the non-realism of optimized samples makes me question this factor as being compelling.  
		      
		    Now there are two solutions (or answers) to this question in my opinion.  
		    First, if the optimization results in plausible real samples or if the privacy canary is a combination of imperceptible perturbations to a real sample then I can imagine a reader being convinced that the result of the privacy audit is meaningful.  
		    However, as I explained earlier, the optimized samples in this work do not seem very realistic.  
		    Second, if the privacy canaries can be used to audit DP-SGD (or other DP-algorithms) more tightly than prior work, then this could also be a very strong motivating factor.  
		    However, the authors do not compare the TPR@low FPR of their optimized canary with prior baselines for DP-SGD in Table 2 and there are large gaps in the empirical epsilon estimated from DP-SGD in Table 3.  
		    Therefore, on both fronts the paper does not satisfactorily solve the motivation.
			- ==Michael== I think the argument here is something like i) privacy is about worst-case leakage of *any* training sample, ii) we don't know if there are any realistic-looking samples that leak a lot of privacy a-priori, iii) there are results showing that even "natural" samples can have very high privacy leakage (e.g., our CCS paper). The key is: if a training procedure protects the privacy of our "unrealistic" canary, then it will also protect the privacy of the "most vulnerable" "realistic" data point, which may or may not be similarly "attackable" as our canaries.
			- Just because we are not auditing DP-SGD, the definition of privacy does not change. wE still need to look at the worst case. One needs to define "natural"
			- The same thing can happen with natural data (Matheew's paper)
			- For DP-SGD auditing the authors mention that due to the computational cost, they can only estimate the epsilon from 20k (or 128 for CIFAR-10) models.  
			  Have the authors tried running the setup of [a], which seemingly only requires 200 models?  
			  
			  **Breadth of Results**  
			  
			  Another weakness of this work I find is that it mainly only focuses on image models and notably leaves out language models (without mentioning why).  
			  I am not sure whether there are specific challenges that prevent the authors from extending this work from image to language models, however the authors do not address these limitations in their work.  
			  Furthermore, an extension to language models can help alleviate the weaknesses of their work in terms of motivation, since strong attacks seem to only be possible when the canaries are repeated (Meeus et al., 2025).
			- ==Michael== We could make two points here: i) Due to the size of LLMs, scaling our method to them requries a lot of additional engineering work; this surpasses the scope of our paper and is future work. ii) "Interesting" privacy leakage in LLMs happens less during pre-training and more during finetuning/RL; the former stage relies on internet data, while the second stage uses pontetially private/proprietary data; while OptiFluence can likely be adapted to such settings, this requires non-trivial novel contributions and is hence more future work.
			  
			  **Prior Work**  
			  
			  Lastly, some prior work specifically on DP-SGD auditing (e.g., [b, c]) has been left out of related work.  
			  Furthermore, [c] actually does seem to optimize adversarial samples for privacy auditing similar to this work but has not been cited or compared to.
			- ==Michael== The workshop link to the paper didn't work but I think it's [this one](https://arxiv.org/abs/2412.01756v2). At a glance, it looks very similar to OptiFluence. But I think what's happening is that they first use a heuristic canary to train all the models during auditing, and then optimize the sample that is used to *query* the models and then calculate DP lower bounds. Our method is much different in that we directly optimize the canary that gets inserted into the trining procedure, and we use the same canary for auditing. But *please check the paper yourself; I didn't quite get it*. In any case, I agree with the reviewer that it's missing from RW.
			  
			  [a] M. S. M. S. Annamalai and E. De Cristofaro. Nearly Tight Black-  
			  Box Auditing of Differentially Private Machine Learning. In  
			  NeurIPS, 2024.  
			  
			  [b] S. Mahloujifar, L. Melis, and K. Chaudhuri. Auditing f-Differential Privacy in One Run.  
			  
			  [c] S. Yoon, W. Jeung, and A. No. Optimizing Adversarial Samples  
			  for Tighter Privacy Auditing in Final Model-Only Settings. In  
			  Statistical Frontiers in LLMs and Foundation Models Workshop  
			  NeurIPS, 2024.
		- #### Key Questions for Authors
		  1. Could the authors better motivate why privacy or data practitioners should care about privacy leakage from optimized canaries?  
		  2. Have the authors tried auditing DP-SGD in other setups [b] where they can potentially showcase the improvements to empirical epsilon estimation?  
		  3. What are the challenges faced when extending the authors' work to language models?  
		  4. How does the authors' work compare with prior work on optimizing canaries [c]?
		- #### Limitations
		    Mostly yes. As mentioned earlier the limitations of the work's applicability to other types of models (e.g., language) have not been discussed.