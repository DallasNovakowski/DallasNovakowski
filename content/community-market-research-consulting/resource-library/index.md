---
# title: "About the Brand"
type: page
# type: docs

show_date: false
show_metadata: false


draft: false
date: ""
# weight: 1
# menu:
#   community-market-research-consulting:
#     parent: community-market-research-consulting
weight: 2
---


<div class="header-with-logo">
  <h1>Resource Library</h1>
  <img src="/community-market-research-consulting/socia_logo.png" alt="Socia Logo" style="height:100px;">
</div>








<style>

/* Base button style */
.cta-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  color: #fff !important;                     /* force white text */
  -webkit-text-fill-color: #fff !important;   /* for iOS / Safari */
  padding: 0.6rem 1rem;
  border-radius: 0.5rem;
  text-decoration: none;
  font-weight: 700;
  min-height: 40px;
  transition: transform .12s ease, box-shadow .12s ease, background-color .18s ease;
  will-change: transform, box-shadow, background-color;
  border: none;
}

/* Specific backgrounds for each CTA */
.cta-button.contact  { background-color: #E8C684; }
.cta-button.examples { background-color: #E3B393; }
.cta-button.library  { background-color: #BDD2D1; }

/* Keep text white in all states */
.cta-button,
.cta-button:link,
.cta-button:visited,
.cta-button:hover,
.cta-button:active,
.cta-button:focus {
  color: #fff !important;
  -webkit-text-fill-color: #fff !important;
  text-decoration: none;
}

/* Hover: slightly lighter background + lift */
.cta-button.contact:hover  { background-color: #F1D79A; }
.cta-button.examples:hover { background-color: #EEC1A5; }
.cta-button.library:hover  { background-color: #C9DDDC; }

.cta-button:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 18px rgba(0,0,0,0.08);
  cursor: pointer;
}

/* Focus accessibility */
.cta-button:focus {
  outline: 3px solid rgba(0,0,0,0.08);
  outline-offset: 2px;
}



/* === Layout container === */

.cta-container {
display: flex;
flex-wrap: wrap;         /* allows stacking on small screens */
align-items: flex-start; /* top align so text doesn't push image down */
justify-content: center; /* center them if wrapping */
column-gap: 2rem;        /* horizontal space between button-wrapper and image */
row-gap: 1rem;           /* vertical spacing when stacked */
}

.cta-container img {
  max-height: 375px;
  height: auto;
  width: auto;
  margin-top: 0rem !important;
  margin-bottom: 0rem !important;
  flex: 0 0 auto;          /* don't let image shrink */
}

.cta-button-wrapper {
  display: flex;
  flex-direction: column;  /* stack button + text vertically */
  align-items: center;     /* keep button centered */
  text-align: center;      /* default: centered text (for small screens) */
  flex: 1 1 300px;         /* allow it to grow wider when wrapping */
  min-width: 220px;        /* prevents squishing too narrow */
}

.cta-button-wrapper .cta-button {
  margin-bottom: 0.5rem;
}

/* description text */
.cta-description {
  font-size: 0.9rem;
  font-style: italic;
  line-height: 1.3;
  color: #333;
  width: 100%;             /* let it expand fully */
  max-width: none;         /* remove the artificial cap */
}

/* On larger screens, left-align the description */
@media (min-width: 768px) {
  .cta-button-wrapper {
    align-items: center;    /* keep button centered */
  }
  .cta-description {
    text-align: left;       /* left-align description text */
  }
}



.header-with-logo {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
}

/* Make sure the h1 takes available space */
.header-with-logo h1 {
  flex: 1 1 auto;
  min-width: 200px;
  margin: 0;
}

/* Image sizing */
.header-with-logo img {
  flex: 0 0 auto;
  margin-top: 0;
}

/* On small screens, stack vertically with logo first */
@media (max-width: 600px) {
  .header-with-logo {
    flex-direction: column;
    align-items: center;  /* center both logo and title */
  }

  .header-with-logo img {
    order: -1;  /* move logo above the h1 */
    margin-bottom: 0.5rem;
  }

  .header-with-logo h1 {
    text-align: center;/* center title on small screens */
        margin-bottom: 2rem;

  }
}
</style>


My intention is for this to be a continually evolving library of materials to help support research and strategy functions.

{{< columns>}}
{{< column >}}

My materials, unless otherwise communicated, are shared under CC BY-SA 4.0 [Creative Commons license - Attribution-ShareAlike 4.0 International Deed](https://creativecommons.org/licenses/by-sa/4.0/) 

{{< /column >}}

{{< column >}}

 Terms:

- **Share** — You may copy and redistribute the material in any medium or format for any purpose, even commercially.
- **Adapt** — You may remix, transform, and build upon the material for any purpose, even commercially. 
- **Attribution** — You must give appropriate credit , provide a link to the license, and indicate if changes were made . You may do so in any reasonable manner, but not in any way that suggests the licensor endorses you or your use.
- **ShareAlike** — If you remix, transform, or build upon the material, you must distribute your contributions under the same license as the original. 

{{< /column >}}
{{< /columns >}}



Many of my research materials (Code Scripts, document templates) are available on [Github](https://github.com/DallasNovakowski)

One that may be of particular relevance to some is my R package `novahelpers` , containing miscellanious helper functions, namely for summarizing and reporting data. Please note that this project is a work in progress, so you are best suited to copying the scripts for your own use and adaptation: https://github.com/DallasNovakowski/novahelpers