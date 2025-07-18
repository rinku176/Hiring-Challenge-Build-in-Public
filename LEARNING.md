**Frog Endless Runner** 

This project was both a fun and challenging experience where I applied my prior knowledge of web development and game design, while also stepping into new territory with vue.js. Here is a reflection on the technical challenges I faced, how I addressed them, the decisions I made, and what I’d do differently next time.

**TECHNICAL CHALLENGES:**
1.  Setting up the Vue Project:
As someone new to vue.js, getting the initial project setup was unexpectedly tricky. Vue’s CLI wasn’t compatible with PowerShell on my system, and I eventually had to switch to the standard Command Prompt to get things working. This took some trial and error, along with referring to official documentation and online forums.

2.  Interpreting the BDD and Adjusting Features:
Some elements described in the brief, such as returning to the previous lily pad on an incorrect jump, posed design issues. For instance, if the lily pad had already scrolled off-screen, this behavior wouldn’t be logical or visually clear. I made deliberate decisions to adjust or omit such features to preserve gameplay clarity and flow.

3.  Transitioning from Web to Game Development:
I had used HTML, CSS, and JavaScript previously during my internship to build web pages, but this was my first time using them for game development. It was eye-opening to reapply familiar tools in a completely different context and learn how to simulate real-time interactivity with them.

**HOW I SOLVED THEM:**
1.  Separation of Logic and Rendering:
Drawing from my experience in traditional game engines, I separated the update loop (game logic) from the render loop (visual updates). This made the system more modular and gave me the flexibility to toggle rendering or pause logic independently without breaking the game.

2.  Handling Game Scaling and Responsiveness:
Making the game playable across different screen sizes, especially mobile, was something I had never handled manually before. Initially, I tried resizing everything relative to screen size, but I later learned to adjust scaling based on gameplay flow and visual clarity. It taught me how layout and game feel go hand-in-hand.

3.  Gameplay Timing and State Management:
The streak-based speed increase (after 10 successful jumps) caused issues with setInterval, especially when the game was paused for streak notifications. To fix this, I implemented a separate timer mechanism that synchronizes with the lily pad generation and handles speed progression without breaking the game loop.

**DESIGN, DECISIONS AND REASONING:**
1.  Feature Adjustments for Clarity:
Rather than implement mechanics that might confuse or frustrate the player (like returning to an off-screen lily pad), I focused on maintaining a smooth, intuitive game loop.

2.  Modular Code Architecture:
Separating core logic from visuals helped with debugging, pausing, and performance optimization. This approach mirrored common best practices in game engine architecture.

3.  Manual Scaling for Gameplay Feel:
Rather than blindly scaling the entire canvas, I tuned object sizes and positioning based on how it felt to play, especially on smaller screens.

**THINGS I WOULD DO DIFFERENT:**
1.  Structure the codebase more cleanly from the beginning with better folder organization and naming conventions.
2.  Make the game mobile-friendly from the start instead of retrofitting responsiveness.
3.  Add audio feedback and more animations to improve the feel of the game.
4.  Introduce procedural elements (e.g. varied obstacles or lily pad types) to increase replayability without relying on handcrafted level design.
5.  Polish the visual style and add dynamic elements to make the game feel less static.

Overall, this project was a fun and challenging journey, and I’m proud of what I was able to build and learn along the way.
