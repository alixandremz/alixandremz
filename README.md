<svg viewBox="0 0 1200 300" xmlns="http://www.w3.org/2000/svg" font-family="'Courier New', Courier, monospace">
  <defs>
    <linearGradient id="sky" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0%" stop-color="#030b06"/>
      <stop offset="65%" stop-color="#0a2416"/>
      <stop offset="100%" stop-color="#0d2c1a"/>
    </linearGradient>

    <radialGradient id="moonGlow" cx="50%" cy="50%" r="50%">
      <stop offset="0%" stop-color="#dff5e6" stop-opacity="0.9"/>
      <stop offset="100%" stop-color="#dff5e6" stop-opacity="0"/>
    </radialGradient>

    <radialGradient id="vignette" cx="50%" cy="45%" r="75%">
      <stop offset="55%" stop-color="#000000" stop-opacity="0"/>
      <stop offset="100%" stop-color="#000000" stop-opacity="0.65"/>
    </radialGradient>

    <filter id="blurSoft"><feGaussianBlur stdDeviation="6"/></filter>
    <filter id="blurGlow"><feGaussianBlur stdDeviation="12"/></filter>
    <filter id="blurSoft2" x="-60%" y="-60%" width="220%" height="220%">
      <feGaussianBlur in="SourceGraphic" stdDeviation="1.6" result="glow"/>
      <feMerge>
        <feMergeNode in="glow"/>
        <feMergeNode in="SourceGraphic"/>
      </feMerge>
    </filter>

    <pattern id="scanlines" width="4" height="4" patternUnits="userSpaceOnUse">
      <rect width="4" height="1" fill="#ffffff" opacity="0.05"/>
    </pattern>

    <style>
      .flicker   { animation: flicker 0.2s steps(2) infinite; }
      .blinkDot  { animation: blinkDot 1s steps(2) infinite; }
      .blinkCurs { animation: blinkDot 0.8s steps(2) infinite; }
      @keyframes flicker { 0%,100% { opacity: 1; } 50% { opacity: 0.93; } }
      @keyframes blinkDot { 0%,49% { opacity: 1; } 50%,100% { opacity: 0; } }
    </style>
  </defs>

  <!-- base sky -->
  <rect width="1200" height="300" fill="url(#sky)"/>

  <!-- moon -->
  <circle cx="1080" cy="55" r="55" fill="url(#moonGlow)" filter="url(#blurGlow)"/>
  <circle cx="1080" cy="55" r="16" fill="#eafff0" opacity="0.85"/>

  <!-- everything below shakes slightly, like handheld footage -->
  <g class="flicker">
    <g id="shakeGroup">
      <animateTransform attributeName="transform" type="translate"
        values="0,0; -2,1; 1,-2; -1,2; 2,-1; 0,1; -1,-1; 0,0"
        dur="1.6s" repeatCount="indefinite"/>

      <!-- back tree layer -->
      <g fill="#04160c" opacity="0.85">
        <polygon points="60,248 90,150 120,248"/>
        <polygon points="150,250 178,160 206,250"/>
        <polygon points="255,246 285,140 315,246"/>
        <polygon points="360,249 386,165 412,249"/>
        <polygon points="470,247 500,155 530,247"/>
        <polygon points="600,250 626,168 652,250"/>
        <polygon points="720,246 750,148 780,246"/>
        <polygon points="840,249 866,162 892,249"/>
        <polygon points="950,247 980,152 1010,247"/>
        <polygon points="1150,249 1176,165 1200,249"/>
      </g>

      <!-- ground -->
      <rect x="0" y="250" width="1200" height="50" fill="#020805"/>
      <ellipse cx="600" cy="255" rx="620" ry="18" fill="#04140b" opacity="0.6"/>

      <!-- front tree layer, darker & bigger -->
      <g fill="#010603">
        <polygon points="20,255 55,120 90,255"/>
        <polygon points="130,258 168,135 206,258"/>
        <polygon points="330,256 372,110 414,256"/>
        <polygon points="560,258 600,128 640,258"/>
        <polygon points="820,257 862,118 904,257"/>
        <polygon points="1000,255 1042,132 1084,255"/>
        <polygon points="1180,258 1200,150 1200,258"/>
      </g>

      <!-- ground mist -->
      <ellipse cx="300" cy="248" rx="180" ry="16" fill="#9fe6b8" opacity="0.05" filter="url(#blurSoft)"/>
      <ellipse cx="750" cy="252" rx="220" ry="18" fill="#9fe6b8" opacity="0.06" filter="url(#blurSoft)"/>

      <!-- ===== running figure ===== -->
      <g id="runnerOuter">
        <animateTransform attributeName="transform" type="translate"
          values="-60,232; 1260,232" dur="7s" repeatCount="indefinite"/>

        <g id="runnerBounce">
          <animateTransform attributeName="transform" type="translate"
            values="0,0; 0,-7; 0,0; 0,-7; 0,0" keyTimes="0;0.25;0.5;0.75;1"
            dur="0.45s" repeatCount="indefinite"/>

          <g stroke="#cdeed9" stroke-width="4.5" stroke-linecap="round" fill="none" filter="url(#blurSoft2)">
            <!-- back arm -->
            <g>
              <animateTransform attributeName="transform" type="rotate"
                values="-32 0 -58; 32 0 -58; -32 0 -58" keyTimes="0;0.5;1"
                dur="0.45s" repeatCount="indefinite"/>
              <line x1="0" y1="-58" x2="0" y2="-30"/>
            </g>
            <!-- back leg -->
            <g>
              <animateTransform attributeName="transform" type="rotate"
                values="34 0 -22; -34 0 -22; 34 0 -22" keyTimes="0;0.5;1"
                dur="0.45s" repeatCount="indefinite"/>
              <line x1="0" y1="-22" x2="0" y2="8"/>
            </g>

            <!-- torso -->
            <line x1="0" y1="-60" x2="0" y2="-22"/>

            <!-- front leg -->
            <g>
              <animateTransform attributeName="transform" type="rotate"
                values="-34 0 -22; 34 0 -22; -34 0 -22" keyTimes="0;0.5;1"
                dur="0.45s" repeatCount="indefinite"/>
              <line x1="0" y1="-22" x2="0" y2="8"/>
            </g>
            <!-- front arm -->
            <g>
              <animateTransform attributeName="transform" type="rotate"
                values="30 0 -58; -30 0 -58; 30 0 -58" keyTimes="0;0.5;1"
                dur="0.45s" repeatCount="indefinite"/>
              <line x1="0" y1="-58" x2="0" y2="-30"/>
            </g>
          </g>

          <!-- head -->
          <circle cx="0" cy="-70" r="9.5" fill="#cdeed9"/>
        </g>
      </g>
    </g>
  </g>

  <!-- vignette -->
  <rect width="1200" height="300" fill="url(#vignette)"/>

  <!-- scanlines -->
  <rect width="1200" height="300" fill="url(#scanlines)"/>

  <!-- REC indicator -->
  <circle class="blinkDot" cx="36" cy="34" r="7" fill="#ff3b3b"/>
  <text x="52" y="40" fill="#ff8080" font-size="18" letter-spacing="2">REC</text>

  <!-- timestamp -->
  <text x="1050" y="284" fill="#7fdc9a" font-size="14" opacity="0.75">00:14:37:09</text>

  <!-- name / role -->
  <text x="36" y="255" fill="#d8ffe0" font-size="26" letter-spacing="1">ALEXANDRE MENEZES<tspan class="blinkCurs">_</tspan></text>
  <text x="36" y="278" fill="#8fdba6" font-size="15" letter-spacing="3" opacity="0.85">FULL STACK DEVELOPER</text>
</svg>

<div align="center">

# Hi, I'm Alexandre Menezes 👋
### Full Stack Developer | Java · Spring Boot · Angular

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/alexandre-menezes-1135a33a1/)
[![Gmail](https://img.shields.io/badge/Email-D14836?style=flat-square&logo=gmail&logoColor=white)](mailto:alixandremenezess@gmail.com)

</div>

---

## 🧑‍💻 About Me

Full Stack Developer with experience building complete applications, from the database to the user interface. I work mainly with **Java/Spring Boot** on the back-end and **Angular** on the front-end, with solid foundations in relational database design and Python automation.

- 🔭 Currently building REST APIs with **Spring Boot**
- 🗄️ Database design and optimization with **MySQL** and **PostgreSQL**
- 🌐 Web interfaces with **Angular** and **JavaScript**
- 🎯 Always aiming for clean code, good practices, and solid architecture
- 📫 Open to new opportunities and collaborations

---

## 🛠️ Tech Stack

<table>
<tr>
<td valign="top" width="33%">

**Back-end**

![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)

</td>
<td valign="top" width="33%">

**Front-end**

![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black)
![Angular](https://img.shields.io/badge/Angular-DD0031?style=flat-square&logo=angular&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat-square&logo=css3&logoColor=white)

</td>
<td valign="top" width="33%">

**Databases**

![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-07405E?style=flat-square&logo=sqlite&logoColor=white)

</td>
</tr>
</table>

**Tools**

![Git](https://img.shields.io/badge/Git-F05032?style=flat-square&logo=git&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)
![Maven](https://img.shields.io/badge/Maven-C71A36?style=flat-square&logo=apachemaven&logoColor=white)
![Postman](https://img.shields.io/badge/Postman-FF6C37?style=flat-square&logo=postman&logoColor=white)
![IntelliJ IDEA](https://img.shields.io/badge/IntelliJ%20IDEA-000000?style=flat-square&logo=intellijidea&logoColor=white)

---
