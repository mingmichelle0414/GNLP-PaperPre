## HOLODECK Pipeline Overview  

- **Floor & Wall Modules:**  
  Develop floor plans, construct wall structures, and select appropriate materials for the floors and walls.  

- **Door & Window Module:**  
  Integrate doorways and windows into the environment.  

- **3D Asset Selection:**  
  Retrieve appropriate 3D assets froms from [Objaverse](https://objaverse.allenai.org/objaverse-1.0).[1].  

- **Constraint-Based Layout Design:**  
  Arrange the assets within the scene by utilizing spatial relational constraints to ensure that the layout of objects is realistic.

<small style="margin-top: 2.3em;">[1]: Objaverse is a comprehensive dataset comprising over 800,000 annotated 3D objects, facilitating various AI and machine learning applications</small>

---

### Objaverse 1.0

<div align="center" style="margin-top: 1.3em;">
  <img src="/holodeck_method/Objaverse.png" width="700">
</div>


---

## Leveraging Objaverse Assets for HOLODECK

- **Asset Curation**: \
    Curating a subset of indoor design assets from Objaverse 1.0.

- **Automatic Annotation**: \
    Annotating assets using GPT-4-Vision with details such as textual descriptions, scale, and canonical views.

- **Library Size**: \
    The library includes 51,464 annotated assets, combining Objaverse and PROCTHOR assets.

- **Optimization for AI2-THOR**: \
    Reducing mesh counts, generating visibility points, and adding colliders to minimize loading time for embodied AI applications.

---

<style>
  ul {
    line-height: 0.8 !important; 
  }
</style>

## HOLODECK Pipeline Overview
- Given any text input, Holodeck generates 3D interactive embodied environments by utilizing a series of specialized modules through multiple rounds of conversation with an LLM (GPT-4).

<div align="center" style="margin-top: 0.4em;">  
  <video controls width="530">  
    <source src="/holodeck_method/pipeline_upscale.mp4" type="video/mp4">  
  </video>  
</div> 

---

## Overall Prompt Design
- **Task Description**: \
    Outlines the context and goals of the task.

- **Output Format**: \
    Specifies the expected structure and type of outputs and  

- **One-shot Example**: \
    A concrete example to assist the LLM’s comprehension of the task.

---

## Floor Plan Prompt
```
    Floor plan Prompt: You are an experienced room designer. 
    Please assist me in crafting a floor plan. Each room is a rectangle. 
    You need to define the four coordinates and specify an appropriate design scheme, including each room’s color, material, and texture.
    Assume the wall thickness is zero. 
    Please ensure that all rooms are connected, not overlapped, and do not contain each other. 
    The output should be in the following format: 
    room name | floor material | wall material | vertices (coordinates). 
    Note: the units for the coordinates are meters.
    For example:
    living room | maple hardwood, matte | light grey drywall, smooth | [(0, 0), (0, 8), (5, 8), (5, 0)] 
    kitchen | white hex tile, glossy | light grey drywall, smooth | [(5, 0), (5, 5), (8, 5), (8, 0)]

    Here are some guidelines for you:
    1. A room’s size range (length or width) is 3m to 8m. The maximum area of a room is 48 m^2.
       Please provide a floor plan within this range and ensure the room is not too small or too large.
    2. It is okay to have one room in the floor plan if you think it is reasonable.
    3. The room name should be unique.
    
    Now, I need a design for {input}.
    Additional requirements: {additional requirements}.
    Your response should be direct and without additional text at the beginning or end.

```

---

## Wall Height Prompt

```
    I am now designing {input}.
    Please help me decide the wall height in meters.
    Answer with a number, for example, 3.0.
    Do not add additional text at the beginning or in the end.
```

---

## Floor & Wall Module

- **Room Definition**:\
    Rooms are rectangles, defined by coordinates. **GPT-4** provides room placement, dimensions, and connectivity.
  
- **Layout Generation**: \
    Produces multi-room layouts with constraints (e.g., minimum area of 9 m²).

- **Material Selection**: \
    Chooses from 236 materials and 148 colors for floor and wall customization.

- **Contextual Customization**: \
    Adapts materials to scene type (e.g., **concrete** for cells, **bricks** for walls).

---

## Floorplan Customizability

- HOLODECK can interpret complicated input and craft reasonable floor plans correspondingly.

<div align="center" style="margin-top: 1.3em;">
  <img src="/holodeck_method/floorplan_customizability.jpg" alt="Customizability" width="850">
</div>

---

## Material Customizability
- HOLODECK can select appropriate floor and wall materials to make the scenes more realistic.
<div align="center" style="margin-top: 1.3em;">
  <img src="/holodeck_method/material_customizability.jpg" alt="Customizability" width="850">
</div>

---



## Doorway & Window Module

- **Room Connections**: \
    Proposes doorways and windows based on LLM queries.

- **Door & Window Types**: \
    40 door styles and 21 window types, each customizable by size, height, quantity, and more.

- **Tailored Designs**: \
    Adapts to specific needs, e.g., wider doors for **wheelchair accessibility** or **floor-to-ceiling windows** in a **sunroom**.

---

## Door & window Customizability

- HOLODECK can adjust the size, quantity, position, etc., of doors & windows based on the input.
<div align="center" style="margin-top: 1.3em;">
  <img src="/holodeck_method/door_winodw_customizability.jpg" alt="Customizability" width="850">
</div>

---

## Object Selection Module

- **Asset Retrieval**: \
    Utilizes the extensive **Objaverse** asset collection to place objects in the scene.

- **LLM Integration**: \
    Proposes objects based on descriptions and dimensions, e.g., "multi-level cat tower, 60×60×180cm."

- **Retrieval Function**[2]:\
    Considers visual and textual similarity and dimensions to ensure the assets match the design.

- **Custom Object Placement**: \
    Places objects on floors, walls, other items, or even ceilings, enhancing scene customization.

<!-- &nbsp; -->
<small style="margin-top: 2em;">[2]They use CLIP to measure the visual similarity, Sentence-BERT for the textual similarity, and 3D bounding box sizes for the dimension. </small>
 
---

## Objects Customizability

- HOLODECK can select and place appropriate floor/wall/small/ceiling objects conditioned on the input.

<div align="center" style="margin-top: 1.3em;">
  <img src="/holodeck_method/asset_customizability.jpg" alt="Customizability" width="850">
</div>

---

## Constraint-based Layout Design Module

- **Objective:**  
  Generates object positioning and orientation for layouts.

- **Challenge with Direct Bounding Boxes:**  
  LLMs can provide absolute bounding box values.  
  This often leads to out-of-bound errors and object collisions.

- **Novel Approach:**  
  Uses LLM to generate **spatial relations** (e.g., "coffee table, in front of, sofa") instead of numerical values.  
  Convert these relations into mathematical conditions.  
  Use an optimization algorithm (with **DFS**) to place objects sequentially.
  
<!-- - **Multiple Valid Layouts:**  
  The probabilistic nature of LLMs yields various valid layouts from the same prompt.
 -->


---

## Constraint-based Layout Design Module

- **Spatial Constraints:**  
  **Types:** Global (edge, middle), Distance (near, far), Position (in front of, side of, above, on top of), Alignment (center aligned), Rotation (face to).

- **LLM** selects a subset of constraints for each object, forming a scene graph for the room. **Handling:**\ 
    a. **Soft constraints:** Allow minor violations when perfect satisfaction isn’t feasible.  
    b. **Hard constraints:** Must be met (no collisions, all objects within boundaries).

- Examples of **Spatial Relational Constraints** generated by LLM and their solutions found by our constraint satisfaction algorithm.
<div align="center" style="margin-top: -0.8em;">
  <img src="/holodeck_method/spatial_constriant.png" width="850">
</div>

<!-- - **Constraint Satisfaction Process (DFS):**  
 a. Choose an **anchor** object (e.g., a bed in a bedroom).  
 b. Sequentially place the remaining objects; a placement is valid only if all hard constraints are satisfied.  
 c. Run the algorithm for a fixed time (30 seconds) to generate multiple candidate layouts.  
 d. Return the layout that satisfies the most constraints. -->

---

## DFS-based Constraint Satisfaction Example

a. DFS solver initiates grids to establish a finite search space.\
b. First explore different placements for the **anchor** object selected by the **LLM**\
c. Optimize the placement for the remaining objects, adhering to the hard constraints, and satisfying as many soft constraints as possible.\
d. Generate multiple solutions, with the final selection meeting the most constraints.

<div align="center" style="margin-top: 1.3em;">
  <img src="/holodeck_method/dfs.png" alt="Customizability" width="850">
</div>

---

## Output Diversity.

- HOLODECK can generate **multiple variants** for the same input with different assets and layouts.
<div align="center" style="margin-top: 1.3em;">
  <img src="/holodeck_method/output_diversity.png" alt="Customizability" width="850">
</div>

---

## Human Evaluation Overview

- **Participants**: 680 graduate students took part in three user studies.
- **Studies**:
  - A comparative study on residential scenes (vs. PROCTHOR), including human evaluations and CLIP Score experiments.\
      a. 120 paired scenes (30 per type: bathroom, bedroom, kitchen, living room)\
      b. The PROCTHOR baseline has access to the same set of Objaverse assets as HOLODECK.
  - An assessment of HOLODECK's performance on 52 diverse scene types.
  - An ablation study on layout design methods, comparing the Spatial Constraint approach with Absolute, Random, and Edge strategies.
- **Outcome**: HOLODECK produces scenes with higher quality and greater diversity.

---

## Comparative Analysis on Residential Scenes (Human Evaluation)

- **Evaluation Criteria**:
  - **Asset Selection**: Faithfulness to scene type
  - **Layout Coherence**: Realism in arrangement
  - **Overall Preference**
- **Results**: HOLODECK preferred in: Asset Selection (59.8%), Layout Coherence (56.9%), Overall Preference (64.4%)

<div align="center" style="margin-top: -0.3em;">
  <img src="/holodeck_method/eval1.png" alt="Customizability" width="500">
</div>

<div align="center" style="margin-top: 0.3em;">
  <small>The pie charts show the distribution of annotator preferences, showing both the percentage and the actual number of annotations favoring each system.</small>
</div>
---

## Comparative Analysis on Residential Scenes (CLIP Score Details)

- **CLIP Score Experiment**:\
    Quantifies the visual coherence between the scene’s top-down view and its scene type prompt ("a top-down view of [scene type]").
- **Reference**: Human-designed scenes from iTHOR are used as the upper bound.
- HOLODECK Significantly outperforms PROCTHOR; closely approaches iTHOR performance.
  
<!-- - **Findings**: \
    HOLODECK's CLIP scores significantly surpass those of PROCTHOR and approach the performance of human-designed iTHOR scenes, confirming its capability to generate visually coherent and accurate scene representations. -->
<!-- - **Findings**:
  1. HOLODECK’s CLIP Scores exceed those of PROCTHOR by significant margins.
  2. Scores closely approach the performance of human-designed iTHOR scenes.
  3. The CLIP Score experiment agrees with our human evaluation.
     -->
<div align="center" style="margin-top: 0.3em;">
  <img src="/holodeck_method/eval2.png" width="530">
</div>

---

## HOLODECK's Performance on Diverse Scenes

- 52 scene types from the MIT Scenes Dataset: \
    Stores (e.g., deli, bakery), Home (e.g., bedroom, dining room), Public Spaces (e.g., museum, locker room), Leisure (e.g., gym, casino), Working Space (e.g., office, meeting room)

<div align="center" style="margin-top: 0em;">
  <img src="/holodeck_method/eval3.1.png" width="830">
</div>


---

# HOLODECK's Performance on Diverse Scenes (Human Evaluation)
- 52 scene types from the MIT Scenes Dataset:
- **Methodology**:
  - Generated five outputs per scene type, totaling 260 scenes.
  - Human annotators rated scenes based on asset selection, layout coherence, and match with scene type (1 to 5 scale).
  - Compared with residential scenes from PROCTHOR and iTHOR.

---

# HOLODECK's Performance on Diverse Scenes (Human Evaluation)
- **Key Findings**:
  - HOLODECK achieved higher human preference scores in 28 out of 52 diverse scenes.
  - Holodeck can generate satisfactory (better than hard-coded rules) outputs for diverse scene types.
  - Demonstrated broad competence and flexibility, outperforming PROCTHOR in many cases.
  - Struggled with scenes requiring complex layouts or unique assets (e.g., dental x-ray machine).

<div align="center" style="margin-top: 0.4em;">
  <img src="/holodeck_method/mit_bar_plot.png" width="880">
</div>


<small> Human evaluation on 52 scene types from MIT Scenes with qualitative examples. The three horizontal lines represent the average score of iTHOR, HOLODECK, and PROCTHOR on four types of residential scenes (bedroom, living room, bathroom and kitchen.) </small>

---

## Ablation Study on Layout Design (Human Evaluation)

- **Layout Design Methods (Baselines)**\
    a. **CONSTRAINT**: HOLODECK’s spatial relational constraint-based layout.\
    b. **ABSOLUTE**: Objects’ absolute coordinates and orientations derived from LLM (similar to LayoutGPT).\
    c. **RANDOM**: Objects are randomly placed within the room without collision.\
    d. **EDGE**: Objects are placed along the room’s walls.
  
<div align="center" style="margin-top: 0.4em;">
  <img src="/holodeck_method/eval4.png" width="580">
</div>

<small style="margin-top: -0.4em;"> HOLODECK’s constraint-based approach outperforms the other methods significantly on bathroom, bedroom and living room. CONSTRAINT and EDGE perform similarly on kitchen, where it is common to align most objects against walls.  </small>

---

## HOLODECK for ObjectNav: Synthesizing Training Environments

<!-- - **Objective:**\ -->
- Holodeck can aid embodied agents in adapting to new scene types and objects during object navigation tasks.

- **NOVELTYTHOR Benchmark:**\
    **92 object types** across 5 categories: Office, Daycare, Music Room, Gym, Arcade.\
    **Scenes**: 100 novel scenes per category generated by HOLODECK.\
    **Task**: Train ObjectNav on HOLODECK scenes and evaluate on NOVELTYTHOR.

- **Model Setup**\
    **Pre-trained** on PROCTHOR-10K, fine-tuned on 100 scenes (50M steps).\
    **PROCTHOR**: \
        **+HOLODECK**: Fine-tune with HOLODECK-generated scenes.\
        **+OBJAVERSE**: Objaverse-enhanced object selection.

    <!-- Use HOLODECK to generate diverse environments for **ObjectNav**, improving agent generalization on novel distributions.-->

---

## Object Navigation in Novel Environments

- Holodeck can aid embodied agents in adapting to new scene types and objects during object navigation tasks.

<div align="center" style="margin-top: 0.4em;">
  <img src="/holodeck_method/objnav.jpg" width="830">
</div>

  
---

- **NoveltyTHOR**, an artist-designed benchmark to evaluate embodied agents in diverse scenes.

<div align="center" style="margin-top: 1.4em;">
  <img src="/holodeck_method/noveltythor1.png" width="700">
</div>

<div align="center" style="margin-top: 0.4em;">
  <img src="/holodeck_method/noveltythor2.png" width="700">
</div>


---

## Results: HOLODECK vs. Baselines

- **HOLODECK** outperforms baselines on **Office**, **Daycare**, and **Music Room**.
   On Gym and Arcade, +HOLODECK and +OBJAVERSE perform similarly.
    <!-- **+HOLODECK** and **+OBJAVERSE** perform similarly on **Gym** and **Arcade**. -->

- **HOLODECK** creates **human-like object placements** (e.g., Music Room with piano, violin, cellos).\
    **PROCTHOR** struggles due to poor object coverage, often performing close to **random**.

- HOLODECK excels in generating **realistic, commonsense layouts**, boosting agent performance on novel environments.

<div align="center" style="margin-top: 0.4em;">
  <img src="/holodeck_method/object_nav_results.png" width="780">
</div>


---

## Conclusion & Future Work  

- **HOLODECK: A New Paradigm for Embodied AI**  
    - Leverages **large language models** to generate **diverse, interactive** environments.  
    - Demonstrated effectiveness through **human evaluation** and **object navigation tasks**.  

- **Future Directions**  
    - Expanding **3D asset library** for richer scene diversity.  
    - Exploring broader **Embodied AI applications**, beyond object navigation.  

🚀 **HOLODECK sets the foundation for scalable, realistic AI training environments.**


---

## Motivation & Importance

<!-- - **Core Claim of Holodeck:**   -->
- Holodeck proposes a fully automated method for creating diverse 3D environments by leveraging large language models (LLMs) and a vast asset library (Objaverse).  

- SciTechDaily’s article, [Not Science Fiction: Researchers Recreate Star Trek’s Holodeck Using AI](https://scitechdaily.com/not-science-fiction-researchers-recreate-star-treks-holodeck-using-ai/), highlights how AI-generated environments could revolutionize training for robotics, AR/VR, and more.

- [The Verge’s article on world modeling](https://www.theverge.com/2025/1/7/24338053/google-deepmind-world-modeling-ai-team-gaming-robot-training) questions whether language-driven methods can fully capture the 3D physical fidelity and interactivity required for real-world tasks.

- **Critiques:**  
    Is reducing manual labor alone sufficient justification for the approach?  
    Can LLMs reliably capture the intricate details of 3D spatial structure and physics necessary for embodied AI?

---

## Limitations of the Proposed Approach

<!-- - **Strengths Highlighted:**   -->
- GPT-4 for common-sense spatial reasoning and semantic understanding.\
Constraint-based optimization helps in achieving coherent object placements.
  
- **Spatial Reasoning & Artifacts:**  
    Heavy reliance on LLMs can sometimes result in hallucinations or imprecise object placements.

<!-- as noted in [PhyScene: Physically Interactable 3D Scene Synthesis](https://arxiv.org/abs/2404.09465).  
 -->
- **Limitations in Diversity:**  
    Current experiments mainly address residential settings, leaving open how the system might handle more diverse or dynamic environments.

- **Critiques:**  
    How well does the system perform with ambiguous, extreme, or atypical prompts?  
    Would a hybrid approach combining LLM outputs with procedural or physics-based rules mitigate these issues?

---

## Evaluation Methodology & Results

-  Extensive human studies (680 participants) indicate a strong preference for Holodeck over procedural baselines like ProcTHOR in terms of asset selection and layout coherence.\
    Zero-shot object navigation on the NoveltyTHOR benchmark shows improved agent performance.

- **Limitations & Biases:**  
    CLIP scores and human ratings focus on aesthetics and layout but may not reflect real-world usability ([Making 3D Scenes Interactive](https://cs231n.stanford.edu/2024/papers/making-3d-scenes-interactive.pdf)).  \
    Evaluations by graduate students may introduce cultural bias, lack diversity in perspectives, and fail to assess adaptability across different tasks and environments.

- **Critiques:**  
    Can additional quantitative metrics (such as collision rate, physical plausibility scores, or simulation-based benchmarks) better validate performance?  \
    How might evaluations be expanded to assess diverse environments and dynamic interactions?


<!-- - **Bias:**  
    Evaluations focus on residential scenes; broader tests (e.g., including dynamic or outdoor environments) and physics-based metrics could provide a more comprehensive evaluation.
 -->

---

# Future Research Directions

- **Dynamic and Physics-Aware Simulation:** Expanding from static to dynamic environments, incorporating real-time physics and interactive object behaviors.

- **Hybrid Pipelines:** Integrating LLM-based scene synthesis with procedural and physics-based models can potentially overcome current limitations.
  
- **Emerging Trends & Additional Resources:**  
  **Hierarchical Inpainting Approaches:**  [Architect: Generating Vivid and Interactive 3D Scenes with Hierarchical 2D Inpainting](https://arxiv.org/abs/2411.09823) proposes a method for refining scene details iteratively, which could enhance realism.\
  **World Modeling and Scalability:** Recent initiatives by companies like DeepMind ([The Verge on world modeling](https://www.theverge.com/2025/1/7/24338053/google-deepmind-world-modeling-ai-team-gaming-robot-training)) signal the importance of scalable and physically accurate world models.


---

# Future Research Directions

- **Challenges in 3D Content Generation:**  
  [Challenges and Opportunities in 3D Content Generation](https://arxiv.org/abs/2405.15335) discusses the need for scalable, high-fidelity 3D synthesis.

- **Critiques:**  
  How can future work improve the real-time interactivity and physical plausibility of AI-generated scenes?  
  What additional domains (such as outdoor environments or dynamic human interactions) should be targeted to broaden the impact in embodied AI?

- **Further Reading and Resources:**  
  [PhyScene (arXiv)](https://arxiv.org/abs/2404.09465)  
  [Challenges in 3D Content Generation (arXiv)](https://arxiv.org/abs/2405.15335)  
  [Google's World Modeling Efforts (The Verge)](https://www.theverge.com/2025/1/7/24338053/google-deepmind-world-modeling-ai-team-gaming-robot-training)

---
transition: slide-up
layout: statement
---

Thanks\
Q & A

