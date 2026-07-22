# Frontend

## Prompt
I'm building a vanilla JS AI Lab website. The frontend is composed of Javascript, HTML and CSS. 
You have access to all libraries applicable to this tech stack. I want to create the home page 
based off the Skeleton layout attached. The colour palette I want must be neutral, light themed
and if any colour is added it must be light pastel.  The web app is split into 3 sections.

The left section provide the menu context and has a header AI Lab. This section is adjustable 
and can be hidden to just show icons. The right section has the same adjustable and hidden 
qualities. The right side allows users to add folders and the folders will be listed below. 
The centre section skeleton page shows the landing page and will display whatever menu context
the user has selected

Show the collapsed sidebar view

Keep the design and layout exactly as is but generate 3 variation with different colour palettes

## Stitch - with Google
Model Selected - 3 Flash

### Themes/ Colour Palettes

1. Mint & Sage (Green) Palette
   This theme uses organic, soft greens to create a calming research environment.
****
/* Mint Theme Styles */
.theme-mint {
--bg-main: #FBFDFB;
--bg-sidebar: #F1F8F4;
--accent-primary: #81C784; /* Sage Green */
--accent-soft: #E0F2F1;    /* Mint Tint */
--text-main: #2D3436;
--text-muted: #636E72;
--border-color: #E8F5E9;
}

.theme-mint .sidebar {
background-color: var(--bg-sidebar);
border-right: 1px solid var(--border-color);
}

.theme-mint .btn-primary {
background-color: var(--accent-primary);
color: white;
border: none;
border-radius: 8px;
padding: 10px 20px;
transition: background 0.3s ease;
}

.theme-mint .btn-primary:hover {
background-color: #66BB6A;
}

.theme-mint .folder-icon {
color: var(--accent-primary);
}
```

2. Azure & Sky (Blue) Palette
   This is a more traditional, clean "Lab" look using light blue tones.

```
/* Azure Blue Theme Styles */
.theme-blue {
--bg-main: #FFFFFF;
--bg-sidebar: #F5F9FF;
--accent-primary: #4A90E2; /* Azure Blue */
--accent-soft: #E1F5FE;    /* Sky Blue */
--text-main: #2C3E50;
--text-muted: #7F8C8D;
--border-color: #E3F2FD;
}

.theme-blue .sidebar {
background-color: var(--bg-sidebar);
border-right: 1px solid var(--border-color);
}

.theme-blue .btn-primary {
background-color: var(--accent-primary);
color: white;
border: none;
border-radius: 8px;
padding: 10px 20px;
transition: background 0.3s ease;
}

.theme-blue .btn-primary:hover {
background-color: #357ABD;
}

.theme-blue .folder-icon {
color: var(--accent-primary);
}
```