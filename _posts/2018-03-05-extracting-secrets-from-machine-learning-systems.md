---
layout: post
title: Extracting secrets from machine learning systems
date: 2018-03-05 14:18:44.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Machine Learning
- Security
meta:
  _oembed_1968bd9c6080ec99efaafcf271c4692e: "{{unknown}}"
  _edit_last: '1'
  _aioseop_opengraph_settings: a:14:{s:32:"aioseop_opengraph_settings_title";s:0:"";s:31:"aioseop_opengraph_settings_desc";s:0:"";s:36:"aioseop_opengraph_settings_customimg";s:0:"";s:37:"aioseop_opengraph_settings_imagewidth";s:0:"";s:38:"aioseop_opengraph_settings_imageheight";s:0:"";s:32:"aioseop_opengraph_settings_video";s:0:"";s:37:"aioseop_opengraph_settings_videowidth";s:0:"";s:38:"aioseop_opengraph_settings_videoheight";s:0:"";s:35:"aioseop_opengraph_settings_category";s:7:"article";s:34:"aioseop_opengraph_settings_section";s:0:"";s:30:"aioseop_opengraph_settings_tag";s:0:"";s:34:"aioseop_opengraph_settings_setcard";s:7:"summary";s:44:"aioseop_opengraph_settings_customimg_twitter";s:0:"";s:44:"aioseop_opengraph_settings_customimg_checker";s:1:"0";}
  _aioseop_description: 'There is an interesting research by guys from Berkeley, Singapore
    and Google about extracting secrets from machine-learning system. '
  _aioseop_title: Extracting secrets from machine-learning systems
  rp4wp_auto_linked: '1'
  _yoast_wpseo_focuskw_text_input: machine learning
  _yoast_wpseo_focuskw: machine learning
  _yoast_wpseo_metadesc: For those who are interested in machine learning and/or security,
    here is a research which discusses extracting secrets from machine learning systems.
  _yoast_wpseo_linkdex: '53'
  _yoast_wpseo_content_score: '30'
  _yoast_wpseo_primary_category: ''
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1618016979'

permalink: "/en/security/extracting-secrets-from-machine-learning-systems.html"
---
For those who are interested in machine learning and/or security, here is a research which discusses extracting secrets from machine learning systems:

[https://arxiv.org/pdf/1802.08232.pdf](https://arxiv.org/pdf/1802.08232.pdf)

The authors say deep learning models can memorize secrets which the training data may contain. Then the authors provide a couple of algorithms which were successfully used for extracting sensitive data from black box machine learning systems. The authors also show that unintended memorization is not the result of overfitting. Finally, they discuss several ways to mitigate the problem.

[According to Mr.&nbsp;Schneier](https://www.schneier.com/blog/archives/2018/03/extracting_secr.html),&nbsp;there is a lot more research to be done here. So good luck :)

