---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

Education
======
* B.Tech. in Computer Science, IIT Bombay, 2014
* M.S. in Computer Science, University of Massachusetts, Amherst, 2019
* Ph.D in Computer Science, Brown University, 2024 (expected)

Work experience
======
* Summer 2018: Data Science Intern
  * Researched the causal effects of user connections in social networks using deep variational auto-encoders for a large data set with millions of connections
  * Selected as top 5 master's students at UMass Amherst, from all the applicants.

* August 2016 - July 2017: Senior Machine Learning Engineer at LinkedIn
  * Led the video classification project. Trained Deep Learning models and deployed them to production to handle more than 1000 fps, using asynchronous processing on GPU clusters
  * Researched the training of Hierarchical Neural Auto-Encoders by introducing Neural Machine Translation type architecture and Tensorflow. Achieved a speed up of 2x
  * Experimented with multiple implementations of a popularity based model, to achieve significant improvements in user engagement. Applied a Thompson sampling based explore-exploit model to engage cold users with companies

* July 2014 - July 2016: Machine Learning Engineer at LinkedIn
  * Developed and maintained a high QPS service, which classified ALL the content created on LinkedIn (millions of entities every day) to filter out spam. Designed various parts of the system: training pipelines for machine learning models, deployment, synchronous and asynchronous classification
  * Worked on a Hackday project to auto-classify advertisements posted on LinkedIn using SVM. The work lead to reduced wait time for ads to go live on LinkedIn.
  

Publications
======
  <ul>{% for post in site.publications %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

Skills
======
* Skill 1
* Skill 2
  * Sub-skill 2.1
  * Sub-skill 2.2
  * Sub-skill 2.3
* Skill 3


Teaching
======
  <ul>{% for post in site.teaching %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>
  
Service and leadership
======
* 
