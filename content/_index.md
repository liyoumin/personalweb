---
# Leave the homepage title empty to use the site title
title: "Youmin Li"
date: 2022-10-24
type: landing

design:
  # Default section spacing
  spacing: "6rem"

sections:
  - block: resume-biography-3
    content:
      # Choose a user profile to display (a folder name within `content/authors/`)
      username: admin
      text: ""
      # Show a call-to-action button under your biography? (optional)
      button:
        text: Download CV
        url: uploads/resume.pdf
    design:
      css_class: dark
      background:
        color: black
        image:
          # Add your image background to `assets/media/`.
          filename: stacked-peaks.svg
          filters:
            brightness: 1.0
          size: cover
          position: center
          parallax: false
  - block: markdown
    content:
      title: '📚 My Research'
      subtitle: ''
      text: |-
       I am interested in interdisciplinary of machine learning, GIS and Remote Sensing application in AgEcon research. I apply a range of qualitative and quantitative methods to comprehensively investigate the role of agricultural science and technology in the Rangeland AgEconomy. I grow up in Inner Mongolia - China, and enjoy the prairie life. That's why I passionate in rangeland AgEcon.
    
        Please reach out to collaborate 😃
    Refer association \: AAEA, SAEA, NAREA, AARES, CARES, EAAE, IFAMA, KAEA.      Student Membershi
    p<img width="468" height="29" alt="image" src="https://github.com/user-attachments/assets/4e64f8dd-947c-4a15-b2a8-3b97a51415f2" />

    design:
      columns: '1'
  - block: collection
    id: papers
    content:
      title: Featured Publications
      filters:
        folders:
          - publication
        featured_only: true
    design:
      view: article-grid
      columns: 2
  - block: collection
    content:
      title: Recent Publications
      text: ""
      filters:
        folders:
          - publication
        exclude_featured: false
    design:
      view: citation
  - block: collection
    id: talks
    content:
      title: Recent & Upcoming Talks
      filters:
        folders:
          - event
    design:
      view: article-grid
      columns: 1
  - block: collection
    id: news
    content:
      title: Recent News
      subtitle: ''
      text: ''
      # Page type to display. E.g. post, talk, publication...
      page_type: post
      # Choose how many pages you would like to display (0 = all pages)
      count: 5
      # Filter on criteria
      filters:
        author: ""
        category: ""
        tag: ""
        exclude_featured: false
        exclude_future: false
        exclude_past: false
        publication_type: ""
      # Choose how many pages you would like to offset by
      offset: 0
      # Page order: descending (desc) or ascending (asc) date.
      order: desc
    design:
      # Choose a layout view
      view: date-title-summary
      # Reduce spacing
      spacing:
        padding: [0, 0, 0, 0]
  - block: cta-card
    demo: true # Only display this section in the Hugo Blox Builder demo site
    content:
      title: Youmin Li
      text: |-

        <a class="Y" href="https://liyoumin.github.io/cv/" data-color-scheme="no-preference: light; light: light; dark: dark;" data-icon="octicon-star" data-size="large" data-show-count="true" aria-label="">Star</a>

        
        From landing pages, second brains, and courses to academic resumés, conferences, and tech blogs.
      button:
        text:
        url: https://liyoumin.github.io/cv/
    design:
      card:
        # Card background color (CSS class)
        css_class: "bg-primary-700"
        css_style: ""
---
