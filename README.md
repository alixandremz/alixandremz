import textwrap
from PIL import ImageFont

FONT_DIR = "/usr/share/fonts/truetype/liberation/"
def font(weight="Regular", size=16):
    name = f"{FONT_DIR}LiberationSans-{weight}.ttf"
    return ImageFont.truetype(name, size)

def wrap(text, size, weight, max_width):
    f = font(weight, size)
    words = text.split()
    lines, cur = [], ""
    for w in words:
        trial = (cur + " " + w).strip()
        if f.getlength(trial) <= max_width or not cur:
            cur = trial
        else:
            lines.append(cur)
            cur = w
    if cur:
        lines.append(cur)
    return lines

def esc(s):
    return s.replace("&", "&amp;").replace("<", "&lt;").replace(">", "&gt;")

# ---------- canvas ----------
W = 1500
MARGIN = 90
CARD_X = MARGIN
CARD_W = W - 2 * MARGIN

FONT_STACK = "-apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Helvetica Neue', 'Liberation Sans', Arial, sans-serif"

parts = []
defs = []
y = 0  # running cursor

# ============================================================
# DEFS: gradients, filters, glass style reused everywhere
# ============================================================
defs.append("""
<defs>
  <linearGradient id="glassFill" x1="0%" y1="0%" x2="100%" y2="100%">
    <stop offset="0%" stop-color="#FFFFFF" stop-opacity="0.34"/>
    <stop offset="45%" stop-color="#EAF3FF" stop-opacity="0.16"/>
    <stop offset="100%" stop-color="#BFD9FF" stop-opacity="0.10"/>
  </linearGradient>
  <linearGradient id="glassStroke" x1="0%" y1="0%" x2="100%" y2="100%">
    <stop offset="0%" stop-color="#FFFFFF" stop-opacity="0.85"/>
    <stop offset="55%" stop-color="#FFFFFF" stop-opacity="0.25"/>
    <stop offset="100%" stop-color="#FFFFFF" stop-opacity="0.05"/>
  </linearGradient>
  <linearGradient id="chipFill" x1="0%" y1="0%" x2="100%" y2="100%">
    <stop offset="0%" stop-color="#FFFFFF" stop-opacity="0.30"/>
    <stop offset="100%" stop-color="#CFE4FF" stop-opacity="0.12"/>
  </linearGradient>
  <linearGradient id="avatarFill" x1="0%" y1="0%" x2="100%" y2="100%">
    <stop offset="0%" stop-color="#EAF4FF" stop-opacity="0.55"/>
    <stop offset="100%" stop-color="#5FA3F5" stop-opacity="0.35"/>
  </linearGradient>
  <linearGradient id="titleFade" x1="0%" y1="0%" x2="0%" y2="100%">
    <stop offset="0%" stop-color="#FFFFFF"/>
    <stop offset="100%" stop-color="#DCEBFF"/>
  </linearGradient>
  <radialGradient id="blobBlue1" cx="50%" cy="50%" r="50%">
    <stop offset="0%" stop-color="#5B9DF7" stop-opacity="0.95"/>
    <stop offset="100%" stop-color="#5B9DF7" stop-opacity="0"/>
  </radialGradient>
  <radialGradient id="blobCyan" cx="50%" cy="50%" r="50%">
    <stop offset="0%" stop-color="#8FD8FF" stop-opacity="0.9"/>
    <stop offset="100%" stop-color="#8FD8FF" stop-opacity="0"/>
  </radialGradient>
  <radialGradient id="blobIndigo" cx="50%" cy="50%" r="50%">
    <stop offset="0%" stop-color="#3555C9" stop-opacity="0.9"/>
    <stop offset="100%" stop-color="#3555C9" stop-opacity="0"/>
  </radialGradient>
  <radialGradient id="blobWhite" cx="50%" cy="50%" r="50%">
    <stop offset="0%" stop-color="#FFFFFF" stop-opacity="0.85"/>
    <stop offset="100%" stop-color="#FFFFFF" stop-opacity="0"/>
  </radialGradient>
  <filter id="blurBig" x="-60%" y="-60%" width="220%" height="220%">
    <feGaussianBlur stdDeviation="70"/>
  </filter>
  <filter id="softShadow" x="-40%" y="-40%" width="180%" height="180%">
    <feDropShadow dx="0" dy="18" stdDeviation="26" flood-color="#0B1E4D" flood-opacity="0.28"/>
  </filter>
  <filter id="chipShadow" x="-60%" y="-60%" width="220%" height="220%">
    <feDropShadow dx="0" dy="6" stdDeviation="8" flood-color="#0B1E4D" flood-opacity="0.18"/>
  </filter>
  <filter id="specBlur" x="-60%" y="-60%" width="220%" height="220%">
    <feGaussianBlur stdDeviation="18"/>
  </filter>
</defs>
""")

def glass_panel(x, y, w, h, r=42):
    """Returns svg for one liquid-glass card (shadow + fill + specular top edge + border)."""
    return f"""
  <g filter="url(#softShadow)">
    <rect x="{x}" y="{y}" width="{w}" height="{h}" rx="{r}" ry="{r}" fill="url(#glassFill)"/>
  </g>
  <rect x="{x}" y="{y}" width="{w}" height="{h}" rx="{r}" ry="{r}" fill="none" stroke="url(#glassStroke)" stroke-width="1.5"/>
  <path d="M {x+r} {y+2} H {x+w-r} " stroke="#FFFFFF" stroke-opacity="0.55" stroke-width="2" stroke-linecap="round" filter="url(#specBlur)"/>
"""

def eyebrow(x, y, text):
    f = font("Bold", 20)
    tw = f.getlength(text)
    pad = 22
    pill_w = tw + pad * 2
    return f"""
  <rect x="{x}" y="{y}" width="{pill_w}" height="42" rx="21" ry="21" fill="#FFFFFF" fill-opacity="0.16" stroke="#FFFFFF" stroke-opacity="0.35" stroke-width="1"/>
  <text x="{x+pill_w/2}" y="{y+27}" text-anchor="middle" font-family="{FONT_STACK}" font-size="20" font-weight="700" letter-spacing="2.5" fill="#EAF3FF">{esc(text)}</text>
""", pill_w

# ============================================================
# BACKGROUND AMBIENT BLOBS (kept away from the outer edges so the
# canvas edge itself stays fully transparent)
# ============================================================
# placeholder, blobs are emitted after we know total H (appended near the end,
# but we draw them first in z-order so we build the string now with an estimate
# and fix cy positions proportionally at the end). To keep it simple we instead
# just spread blobs relative to fractions of final height, computed later.

# ============================================================
# HEADER
# ============================================================
y += 70
avatar_r = 92
avatar_cx = W / 2
avatar_cy = y + avatar_r

parts.append(f"""
<g filter="url(#softShadow)">
  <circle cx="{avatar_cx}" cy="{avatar_cy}" r="{avatar_r}" fill="url(#avatarFill)"/>
</g>
<circle cx="{avatar_cx}" cy="{avatar_cy}" r="{avatar_r}" fill="none" stroke="url(#glassStroke)" stroke-width="2"/>
<text x="{avatar_cx}" y="{avatar_cy+18}" text-anchor="middle" font-family="{FONT_STACK}" font-size="76" fill="#FFFFFF">&lt;/&gt;</text>
<g filter="url(#chipShadow)">
  <circle cx="{avatar_cx+avatar_r*0.72}" cy="{avatar_cy+avatar_r*0.72}" r="34" fill="#0E2A63" fill-opacity="0.55" stroke="#FFFFFF" stroke-opacity="0.5" stroke-width="1.5"/>
</g>
<text x="{avatar_cx+avatar_r*0.72}" y="{avatar_cy+avatar_r*0.72+11}" text-anchor="middle" font-family="{FONT_STACK}" font-size="30">🇧🇷</text>
""")
y = avatar_cy + avatar_r + 46

title = "Hello, everyone"
f_title = font("Bold", 74)
parts.append(f"""
<text x="{W/2 - 34}" y="{y+60}" text-anchor="middle" font-family="{FONT_STACK}" font-size="74" font-weight="700" letter-spacing="-1.5" fill="url(#titleFade)">{esc(title)}</text>
<text x="{W/2 + f_title.getlength(title)/2 + 24}" y="{y+60}" text-anchor="middle" font-family="{FONT_STACK}" font-size="62">👋</text>
""")
y += 100

subtitle = "Technology Student from Brazil  ·  Backend-leaning Developer"
parts.append(f"""
<text x="{W/2}" y="{y+14}" text-anchor="middle" font-family="{FONT_STACK}" font-size="29" font-weight="400" fill="#EAF3FF" fill-opacity="0.88">{esc(subtitle)}</text>
""")
y += 90

# ============================================================
# ABOUT ME
# ============================================================
about_pad = 56
eb_svg, eb_w = eyebrow(CARD_X + about_pad, y + about_pad - 6, "ABOUT ME")

about_text = ("I'm a technology student from Brazil, focused on building solid software "
              "foundations. I enjoy backend development, working with relational databases, "
              "and understanding how systems fit together \u2014 from the Linux command line up "
              "to the application layer. Always learning, always building something new.")
about_lines = wrap(about_text, 27, "Regular", CARD_W - about_pad * 2)
line_h = 42
about_text_h = line_h * len(about_lines)
about_h = about_pad + 42 + 28 + about_text_h + about_pad

about_svg = glass_panel(CARD_X, y, CARD_W, about_h)
about_svg += eb_svg
ty = y + about_pad + 42 + 44
for line in about_lines:
    about_svg += f'<text x="{CARD_X+about_pad}" y="{ty}" font-family="{FONT_STACK}" font-size="27" fill="#F3F8FF" fill-opacity="0.94">{esc(line)}</text>\n'
    ty += line_h
parts.append(about_svg)
y += about_h + 56

# ============================================================
# SKILLS
# ============================================================
skills = [
    ("🐍", "Python"), ("☕", "Java"), ("🌱", "Spring Boot"), ("🗄️", "SQL"),
    ("🐘", "PostgreSQL"), ("🐬", "MySQL"), ("🐧", "Linux"), (">_", "Bash / Shell"),
    ("🐦", "Swift"), ("✨", "SwiftUI"), ("🔀", "Git"), ("🐳", "Docker"),
    ("🧩", "VS Code"), ("💡", "IntelliJ IDEA"),
]

sk_pad = 56
eb_svg2, _ = eyebrow(CARD_X + sk_pad, 0, "SKILLS & TOOLS")  # y fixed later

chip_font = font("Bold", 23)
chip_h = 62
chip_gap_x = 18
chip_gap_y = 18
chip_pad_x = 26
icon_w = 40

rows = []
row = []
row_w = 0
max_row_w = CARD_W - sk_pad * 2
for icon, label in skills:
    label_w = chip_font.getlength(label)
    chip_w = icon_w + label_w + chip_pad_x * 2
    if row and row_w + chip_gap_x + chip_w > max_row_w:
        rows.append((row, row_w))
        row = []
        row_w = 0
    if row:
        row_w += chip_gap_x
    row.append((icon, label, chip_w))
    row_w += chip_w
if row:
    rows.append((row, row_w))

skills_inner_h = len(rows) * chip_h + (len(rows) - 1) * chip_gap_y
skills_h = sk_pad + 42 + 30 + skills_inner_h + sk_pad

skills_svg = glass_panel(CARD_X, y, CARD_W, skills_h)
eb_svg2, _ = eyebrow(CARD_X + sk_pad, y + sk_pad - 6, "SKILLS & TOOLS")
skills_svg += eb_svg2

cy = y + sk_pad + 42 + 30
for row, row_w in rows:
    cx = CARD_X + sk_pad + (max_row_w - row_w) / 2  # center each row
    for icon, label, chip_w in row:
        skills_svg += f"""
  <g filter="url(#chipShadow)">
    <rect x="{cx}" y="{cy}" width="{chip_w}" height="{chip_h}" rx="{chip_h/2}" ry="{chip_h/2}" fill="url(#chipFill)" stroke="#FFFFFF" stroke-opacity="0.4" stroke-width="1.2"/>
  </g>
  <text x="{cx+chip_pad_x+2}" y="{cy+chip_h/2+9}" font-family="{FONT_STACK}" font-size="24" fill="#FFFFFF">{esc(icon)}</text>
  <text x="{cx+chip_pad_x+icon_w}" y="{cy+chip_h/2+8}" font-family="{FONT_STACK}" font-size="23" font-weight="700" fill="#FFFFFF">{esc(label)}</text>
"""
        cx += chip_w + chip_gap_x
    cy += chip_h + chip_gap_y

parts.append(skills_svg)
y += skills_h + 56

# ============================================================
# PROJECTS
# ============================================================
pr_pad = 56
eb_svg3, _ = eyebrow(CARD_X + pr_pad, y + pr_pad - 6, "FEATURED PROJECTS")

projects = [
    ("Learning SQL with Pokémon", "SQL  ·  Practice Project",
     "A hands-on project for practicing SQL by building and querying a "
     "Pok\u00e9mon-themed database \u2014 writing joins, filters and aggregations "
     "over real Pok\u00e9mon data."),
    ("Cratovia", "Java  ·  Spring Boot",
     "A personal project for strengthening backend fundamentals with Java and "
     "Spring Boot, exploring clean architecture and RESTful API design."),
]

gap = 40
sub_w = (CARD_W - pr_pad * 2 - gap) / 2
proj_lines = [wrap(desc, 21, "Regular", sub_w - 56) for _, _, desc in projects]
max_lines = max(len(l) for l in proj_lines)
sub_h = 46 + 34 + max_lines * 32 + 40 + 30

projects_h = pr_pad + 42 + 30 + sub_h + pr_pad

proj_svg = glass_panel(CARD_X, y, CARD_W, projects_h)
proj_svg += eb_svg3

sub_y = y + pr_pad + 42 + 30
for i, (title, tag, desc) in enumerate(projects):
    sx = CARD_X + pr_pad + i * (sub_w + gap)
    proj_svg += f"""
  <rect x="{sx}" y="{sub_y}" width="{sub_w}" height="{sub_h}" rx="30" ry="30" fill="#FFFFFF" fill-opacity="0.10" stroke="#FFFFFF" stroke-opacity="0.30" stroke-width="1.2"/>
  <text x="{sx+34}" y="{sub_y+52}" font-family="{FONT_STACK}" font-size="30" font-weight="700" fill="#FFFFFF">{esc(title)}</text>
  <text x="{sx+34}" y="{sub_y+86}" font-family="{FONT_STACK}" font-size="20" font-weight="700" letter-spacing="1" fill="#BFE0FF">{esc(tag)}</text>
"""
    ty = sub_y + 86 + 40
    for line in proj_lines[i]:
        proj_svg += f'<text x="{sx+34}" y="{ty}" font-family="{FONT_STACK}" font-size="21" fill="#F3F8FF" fill-opacity="0.92">{esc(line)}</text>\n'
        ty += 32

parts.append(proj_svg)
y += projects_h + 70

# ============================================================
# FOOTER
# ============================================================
parts.append(f"""
<line x1="{W/2-160}" y1="{y}" x2="{W/2+160}" y2="{y}" stroke="#FFFFFF" stroke-opacity="0.35" stroke-width="1.5"/>
<text x="{W/2}" y="{y+54}" text-anchor="middle" font-family="{FONT_STACK}" font-size="26" font-weight="700" fill="#FFFFFF" fill-opacity="0.9">Thanks for stopping by 🚀</text>
""")
y += 110

H = int(y)

# ---------- ambient blobs (placed proportionally now that H is known) ----------
blob_svg = f"""
<circle cx="{W*0.18}" cy="{H*0.10}" r="360" fill="url(#blobCyan)" filter="url(#blurBig)"/>
<circle cx="{W*0.85}" cy="{H*0.22}" r="420" fill="url(#blobBlue1)" filter="url(#blurBig)"/>
<circle cx="{W*0.12}" cy="{H*0.55}" r="380" fill="url(#blobIndigo)" filter="url(#blurBig)"/>
<circle cx="{W*0.90}" cy="{H*0.62}" r="360" fill="url(#blobWhite)" filter="url(#blurBig)" opacity="0.5"/>
<circle cx="{W*0.30}" cy="{H*0.90}" r="420" fill="url(#blobBlue1)" filter="url(#blurBig)" opacity="0.8"/>
<circle cx="{W*0.80}" cy="{H*0.95}" r="360" fill="url(#blobCyan)" filter="url(#blurBig)" opacity="0.6"/>
"""

svg = f"""<svg width="{W}" height="{H}" viewBox="0 0 {W} {H}" xmlns="http://www.w3.org/2000/svg">
{''.join(defs)}
<g>
{blob_svg}
</g>
{''.join(parts)}
</svg>"""

with open("/home/claude/readme_banner/profile_banner.svg", "w") as f:
    f.write(svg)

print("W,H =", W, H)
print("about_h", about_h, "skills_h", skills_h, "rows", len(rows), "projects_h", projects_h)
