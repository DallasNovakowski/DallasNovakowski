---
# title: "Example Products"
type: page
# type: docs
draft: false
date: false
lastmod: false
show_date: false
show_metadata: false


# weight: 1
# menu:
#   community-market-research-consulting:
#     parent: community-market-research-consulting
# weight: 2

header:
  logo: "/community-market-research-consulting/socia_logo.png"
---


<div class="header-with-logo">
  <h1>Example Products</h1>
  <img src="/community-market-research-consulting/socia_logo.png" alt="Socia Logo" style="height:100px;">
</div>




This page offers some demos of what some of my work looks like. Please note that these products do not contain real data, instead being simulated for the purposes of demonstration. The materials here can be accessed via my [github site](https://github.com/DallasNovakowski/socia_demo)


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

.page-header,
.article-container,
.docs-content > *:first-child {
  margin-top: 0 !important;
  padding-top: 0 !important;
}

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
  margin-top: 0;
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




### My Preferred Tech and Tools

My primary tools and software used revolves around the the programming language R, and its related applications and packages, such as Rstudio (for user interface), ggplot (for data visualization) Shiny (for dashboarding), and Quarto (for generating reports). This system has the particular benefit of being open source, meaning that the software is free to use and edit, so that advanced methods are constantly being added. However, I am also proficient in other software and tools as needed (e.g. Tableau, PowerBI, Python, Excel, SQL, PowerQuery).

## Methods for Visualizing and Reporting Data


<div style="padding: .5rem; border-radius: 0.5rem; margin-top: 1.5rem; margin-bottom: 1.5rem;">
  <div class="cta-container">

  <!-- Button + description -->
  <div class="cta-button-wrapper">
    <a href="https://gzlvni-dallas-novakowski.shinyapps.io/socia_demo/" class="cta-button library">Click to view Demo Dashboard</a>
    <div class="cta-description">
      Dashboards have become popular for good reason; they give users agency to customize the analyses available, selecting particular target measures, while grouping and filtering to address their particular needs. 
      <br> <br>
  Working in close collaboration with your team, we will make a dashboard to suit your unique needs.
    </div>
  </div>

  <!-- Image -->
  <a href="https://gzlvni-dallas-novakowski.shinyapps.io/socia_demo/" target="_blank">
    <img src="dashboard.png" alt="A screenshot of Socia's demo dashboard"/>
  </a>

  </div>
</div>



<div style="padding: .5rem; border-radius: 0.5rem; margin:0rem;">
  <div class="cta-container">

  <!-- Button + description -->
  <div class="cta-button-wrapper">
    <a href="/uploads/socia_report_demo.pdf" class="cta-button examples">Click to View Static Reports</a>
    <div class="cta-description">
      Research and data analysis is about more than just showing data - it's about telling a story. Often, teams need a curated report to show them exactly what is going on, and what it means for you. <br> <br> I work hard to learn your exact organizational context and priorities, and create digestible reports that do the heavy lifting of selecting relevant groups and filters, to show you exactly what you need to know, and how it can be used. 

  </div>
  </div>


  <!-- Image -->
  <a href="/uploads/socia_report_demo.pdf" target="_blank">
    <img src="static_report.png" alt="A screenshot of Socia's demo dashboard"/>
  </a>

  </div>
</div>




{{< columns>}}
{{< column >}}

{{< /column >}}

{{< column >}}



{{< /column >}}
{{< /columns >}}

