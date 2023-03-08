---
# Mr. Green Jekyll Theme (https://github.com/MrGreensWorkshop/MrGreen-JekyllTheme)
# Copyright (c) 2022 Mr. Green's Workshop https://www.MrGreensWorkshop.com
# Licensed under MIT

layout: default
# About page
---
{%- include multi_lng/get-lng-by-url.liquid -%}
{%- assign lng = get_lng -%}
<div class="multipurpose-container about-container">
  <div class="row about-main">
    <div class="col-md-3 about-img">
      <img src="{{ page.img }}" alt="">
    </div>
    <div class="col-md-9 about-header">
      <h1 translate="no">{{ site.data.owner[lng].brand }}</h1>
      <div class="meta-container">
        {%- assign about_title = site.data.owner[lng].about.sub_title | replace: site.data.conf.main.sample_replace -%}
        {%- if site.data.owner[lng].about.sub_title %}
          <p class="sub-title">
            {%- if site.data.conf.others.about.sub_title_icon %}<i class="{{ 'fa-fw ' }}{{ site.data.conf.others.about.sub_title_icon }}" aria-hidden="true"></i>{% endif -%}
            &nbsp;{{ about_title }}
          </p>
        {% endif -%}
        {%- assign tmp_obj =  site.data.owner[lng].contacts | where_exp: "item", "item.email != nil" | first -%}
        {%- assign email = tmp_obj['email'] -%}
        {%- if site.data.conf.others.about.show_email and email %}
          {%- assign _email = email | split: '@' %}
          <p class="email">
            <a href="javascript:void(0);" onclick="setAddress('{{ _email[0] }}', '{{ _email[1] }}');">
              {%- if site.data.conf.others.about.email_icon %}<i class="{{ 'fa-fw ' }}{{ site.data.conf.others.about.email_icon }}"></i>{% endif -%}
              &nbsp;{{ site.data.lang[lng].about.email_title }}
            </a>
          </p>
        {% endif -%}
        {%- if site.data.conf.others.about.show_contacts and site.data.owner[lng].contacts.size > 0 %}
          {% include default/nav/contact-links.html -%}
        {% endif -%}
      </div>
    </div>
  </div>
  <div class="row about-divider">
    <hr>
  </div>
  <div class="row">
    <div class="col-md-12">
      <div class="about-msg markdown-style">
        {{ 데이터를 요리조리 다루어 사용자에게 필요한 것을 만드는 박수진입니다. 창의력 수학 교재 개발과 분석한 경험을 토대로 주어진   데이터를 어떻게 가치 있게 만드는 것에 고민하는 것을 좋아합니다. 데이터 분석과 예측, 자연어 처리에 대해 호기심이 많아 끊임없이 배우는 것을 즐깁니다. }}
        {%- if site.data.conf.main.contact_form.enable and site.data.conf.others.about.show_contact_form_button %}
          <a href="javascript:void(0);" class="btn-base " onclick="ContactForm.show();" role="button">{{ site.data.lang[lng].contact_form.button_name }}</a>
        {% endif -%}
      </div>
    </div>
  </div>
</div>
