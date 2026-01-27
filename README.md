<h1 align="center">OccluReStyle3D: Temporally Consistent Occlusion-Aware Stylization </h1> <p align="center"> <em>Extending scene-level appearance transfer ResStyle3D toward dynamic videos with human occlusion — motivating Temporally Consistent Occlusion-Aware Stylization.</em> </p> <p align="center"> <a href="https://restyle3d.github.io/" target="_blank">ReStyle3D (Paper/Project)</a> • <a href="https://shangchenzhou.com/projects/ProPainter/" target="_blank">ProPainter</a> • <a href="https://aidemos.meta.com/segment-anything" target="_blank">Segment Anything (SAM)</a> </p> <hr/> <!-- TODO: Add hero demo video (GitHub user-attachments link) --> <!-- https://github.com/user-attachments/assets/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX --> 
<h2>🌐 Overview</h2> <p> <b>ReStyle3D</b> introduces <b>SceneTransfer</b>, a task of compositionally transferring multi-object appearance from a <em>single style image</em> to a <b>3D scene</b> captured via multi-view images or video. The key promise is to preserve scene structure while re-texturing semantic regions using <b>semantic correspondences</b> across views. </p> <p> This repository explores a practical and currently underdeveloped direction: <b>scene-level appearance transfer in dynamic videos with human presence</b>, where people move through the scene and partially occlude background surfaces. </p> <hr/> 
<h2>Task: Scene Style Transfer</h2> <p> <b>Syle Transfer</b> aims to transfer multi-object appearance from a single style reference image onto a reconstructed 3D scene, so that rendered or reprojected views are both: </p> <ul> <li><b>Semantically consistent</b>: objects receive appropriate appearance changes conditioned on their semantic identity.</li> <li><b>View-consistent</b>: stylization remains coherent across viewpoints (multi-view images / video).</li> <li><b>Structure-preserving</b>: geometry and layout remain stable while appearance changes.</li> </ul> <hr/> <h2>Research Question</h2> <p> Can we extend ReStyle3D to <b>dynamic scenes involving humans</b> — i.e., <b>Temporally Consistent Occlusion-Aware Stylization</b> from videos? </p> <hr/> 
<h2>💡 Motivation</h2> <p> Dynamic scenes with human presence are central to interactive applications such as: <b>telepresence</b>, <b>gaming</b>, <b>film production</b>, and <b>virtual try-on</b>. In these settings, a system should stylize the environment while handling: </p> <ul> <li><b>Non-rigid motion</b> (people moving, articulating, deforming)</li> <li><b>Partial occlusions</b> (humans frequently block background regions)</li> <li><b>Temporal coherence</b> (appearance should remain stable frame-to-frame)</li> </ul> <p> However, <b>ReStyle3D is inherently designed for static scenes</b>. When applied to videos with humans, it tends to: (1) treat humans as just another semantic category and stylize them directly, and (2) break temporal consistency under occlusion and motion. </p> <hr/>

<h2>Limitation: Why ReStyle3D Does Not Trivially Extend to Dynamic Scenes</h2>

<p>
In dynamic videos, humans introduce <b>time-varying visibility</b> over the same scene surfaces. This creates a strict requirement:
the stylized background must remain <b>stable under occlusion and disocclusion</b>—i.e., when the same surface disappears behind a person and later reappears.
</p>

<p>
<b>Video Example (Failure Case).</b> The clip below highlights the core issue: as a moving person occludes the background,
ReStyle3D’s stylization becomes unstable—causing <b>flicker</b>, <b>semantic leakage onto the human</b>, and <b>inconsistency</b> in re-exposed regions.
</p>



https://github.com/user-attachments/assets/66ce0deb-c363-4bd2-b702-ea1685979a8c





<ul>
  <li><b>Human occlusion breaks correspondences:</b> background pixels disappear and reappear, complicating stable appearance transfer.</li>
  <li><b>Humans get stylized unintentionally:</b> ReStyle3D transfers style to humans as part of semantic categories, instead of treating them as dynamic occluders.</li>
  <li><b>Temporal consistency is not guaranteed:</b> stylization can flicker across frames, especially near occlusion boundaries.</li>
</ul>

<p>
Conclusion: <b>ReStyle3D cannot be trivially extended to dynamic scenes involving human occlusion</b> without additional modeling.
A dedicated <b>temporally consistent, occlusion-aware stylization module</b> is required.
</p>

<hr/>


 <h2>OccluReStyle3D Baseline: Inpaint-Then-Stylize (Background) + Superimpose Humans</h2> <p> To stress-test the dynamic-occlusion setting and establish a simple baseline, we build an <b>inpaint-based decomposition</b> pipeline: </p> <ol> <li><b>Human Masking:</b> Use <b>SAM</b> to segment humans per frame and obtain a human-specific occlusion mask.</li> <li><b>Video Inpainting:</b> Use <b>ProPainter</b> to inpaint masked regions, producing a “human-removed” background video.</li> <li><b>Style the Background:</b> Apply <b>ReStyle3D</b> / SceneTransfer-style appearance transfer to the inpainted background video (treated as a static scene proxy).</li> <li><b>Superimpose Humans:</b> Composite original humans back onto the stylized background to obtain a final “styled scene with humans”.</li> </ol> <p> This baseline is intentionally simple: it isolates the background stylization problem by attempting to remove the dynamic occluder (human) first. </p> <!-- TODO: Add pipeline image --> <!-- <img width="1028" alt="pipeline" src="https://github.com/user-attachments/assets/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" /> --> <hr/> <h2>🎥 Qualitative Results</h2> <p> We will add videos demonstrating: </p> <ul> <li>Original video (dynamic scene with humans)</li> <li>SAM masks</li> <li>ProPainter inpainted background</li> <li>Stylized background (ReStyle3D on inpainted video)</li> <li>Final composite (stylized background + original humans)</li> </ul> <!-- TODO: Add qualitative demo videos --> <!-- https://github.com/user-attachments/assets/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX --> <hr/> <h2>🚧 Known Failure Modes of the Baseline</h2> <p> While “inpaint + stylize + composite” is a useful diagnostic, it introduces substantial artifacts: </p> <ul> <li><b>Temporally inconsistent lighting:</b> background illumination and shading may not match humans consistently over time.</li> <li><b>Unrealistic human–scene interaction:</b> contact regions (feet on ground, hands on objects) look physically incoherent after compositing.</li> <li><b>Artifacts and aberrations:</b> inpainting errors and stylization boundary issues accumulate around occlusion edges.</li> </ul> <p> These issues reinforce the core point: <b>ReStyle3D cannot be trivially extended to dynamic scenes involving human occlusion</b>. A <b>specialized module for Temporally Consistent Occlusion-Aware Stylization</b> is required. </p> <hr/> <h2>🧩 Why a Specialized Module Is Needed</h2> <p> A robust solution for dynamic, human-occluded scenes likely requires: </p> <ul> <li><b>Explicit occlusion reasoning</b> (who occludes what, and when)</li> <li><b>Temporal propagation</b> of appearance/texture in a way that is stable under disocclusion</li> <li><b>Human-aware compositing</b> with consistent lighting and physically plausible boundaries</li> <li><b>Interaction-aware constraints</b> to preserve contact realism at human–scene interfaces</li> </ul> <hr/> <h2>▶️ Running the Baseline (Placeholder)</h2> <p> We will provide scripts and exact commands. High-level placeholders: </p>

# 1) Generate human masks (SAM)
python scripts/run_sam_human_mask.py --video <input.mp4> --out_dir outputs/masks

# 2) Inpaint background (ProPainter)
python scripts/run_propainter.py --video <input.mp4> --mask_dir outputs/masks --out_dir outputs/inpainted_bg

# 3) Style-transfer background (ReStyle3D / SceneTransfer)
python scripts/run_restyle3d.py --input outputs/inpainted_bg --style <style.jpg> --out_dir outputs/stylized_bg

# 4) Composite humans back
python scripts/composite_humans.py --orig_video <input.mp4> --mask_dir outputs/masks --bg_video outputs/stylized_bg --out outputs/final.mp4

<hr/> <h2>📚 References</h2> <ul> <li><a href="#" target="_blank">ReStyle3D: Scene-Level Appearance Transfer with Semantic Correspondences</a></li> <li><a href="#" target="_blank">ProPainter: Improving Propagation and Transformer for Video Inpainting</a></li> <li><a href="#" target="_blank">Segment Anything (SAM)</a></li> </ul> <hr/> <h2>🤝 Acknowledgements</h2> <p> This project builds on open-source and published work including ReStyle3D, ProPainter, and Segment Anything (SAM). All credit for original methods belongs to their respective authors. </p> <hr/>
