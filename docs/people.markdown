---
layout: page
title: People
permalink: /people/
---

{%- assign members = site.data.people | where_exp: "p", "p.alumni != true" -%}
{%- assign alumni = site.data.people | where_exp: "p", "p.alumni == true" -%}

<div class="people-grid">
  {%- for person in members %}
  {%- assign name_parts = person.name | split: " " %}
  <a class="person-tile" href="#person-{{ person.id }}" aria-label="Read more about {{ person.name }}">
    {%- if person.photo %}
    <img class="person-tile__photo" src="{{ person.photo | relative_url }}" alt="{{ person.name }}" loading="lazy">
    {%- else %}
    <span class="person-tile__photo person-tile__photo--placeholder" aria-hidden="true">{{ name_parts[0] | slice: 0 }}{% if name_parts.size > 1 %}{{ name_parts[1] | slice: 0 }}{% endif %}</span>
    {%- endif %}
    {%- comment %} Given name on the first line, the rest of the name on the second. {% endcomment %}
    <span class="person-tile__name">{{ name_parts | first }}<br>{{ person.name | remove_first: name_parts[0] | strip }}</span>
    <span class="person-tile__role">{{ person.role }}</span>
  </a>
  {%- endfor %}
</div>

{%- for person in members %}
<div class="person-modal full-bleed" id="person-{{ person.id }}" role="dialog" aria-modal="true" aria-labelledby="person-{{ person.id }}-name">
  <a class="person-modal__backdrop" href="#" tabindex="-1" aria-hidden="true"></a>
  <div class="person-modal__card">
    <a class="person-modal__close" href="#" aria-label="Close">&times;</a>
    <div class="person-modal__head">
      {%- if person.photo %}
      <img class="person-modal__photo" src="{{ person.photo | relative_url }}" alt="{{ person.name }}" loading="lazy">
      {%- endif %}
      <div>
        <h2 class="person-modal__name" id="person-{{ person.id }}-name">{{ person.name }}</h2>
        {%- for title in person.titles %}
        <p class="person-modal__title">{{ title }}</p>
        {%- endfor %}
      </div>
    </div>
    {%- for para in person.bio %}
    <p>{{ para }}</p>
    {%- endfor %}
    {%- if person.education %}
    <p><strong>Education:</strong> {{ person.education }}</p>
    {%- endif %}
    {%- capture contact -%}
      {%- if person.email -%}{{ person.email }}{%- endif -%}
      {%- for link in person.links -%}
        {%- if person.email or forloop.first == false %} | {% endif -%}
        <a href="{{ link.url }}">{{ link.text }}</a>
      {%- endfor -%}
    {%- endcapture -%}
    {%- assign contact = contact | strip %}
    {%- if contact != "" %}
    <p><strong>Contact:</strong> {{ contact }}</p>
    {%- endif %}
  </div>
</div>
{%- endfor %}

{%- if alumni.size > 0 %}
<h2 class="alumni-heading" id="alumni">Alumni</h2>

<ul class="alumni-list">
  {%- for person in alumni %}
  <li class="alumni-item">
    <span class="alumni-name">{{ person.name }}</span>
    {%- assign alum_title = person.titles | join: " · " | default: person.role -%}
    {%- if alum_title %}<span class="alumni-role">{{ alum_title }}</span>{% endif %}
    {%- if person.current %}
    <span class="alumni-current">Now: {% if person.current.url %}<a href="{{ person.current.url }}">{{ person.current.text }}</a>{% else %}{{ person.current.text }}{% endif %}</span>
    {%- endif %}
  </li>
  {%- endfor %}
</ul>
{%- endif %}

<script>
  // Progressive enhancement over the :target CSS, which already opens and
  // closes these cards with no JavaScript at all. This adds Esc to close,
  // avoids the jump-scroll that following an anchor would cause, and moves
  // focus into the card and back to the tile afterwards.
  (function () {
    var lastTrigger = null;

    function openModal(id, trigger) {
      var modal = document.getElementById(id);
      if (!modal) return;
      lastTrigger = trigger || null;
      // Record it in the URL so the card is linkable and Back closes it,
      // without letting the browser scroll to the anchor.
      if (history.pushState) {
        history.pushState(null, '', '#' + id);
      } else {
        location.hash = id;
      }
      sync();
      var card = modal.querySelector('.person-modal__close');
      if (card) card.focus();
    }

    function closeModal() {
      if (history.pushState) {
        history.pushState(null, '', location.pathname + location.search);
      } else {
        location.hash = '';
      }
      sync();
      if (lastTrigger) {
        lastTrigger.focus();
        lastTrigger = null;
      }
    }

    // Keep the open/closed state (and aria-hidden) in step with the hash.
    function sync() {
      var current = location.hash.replace('#', '');
      var modals = document.querySelectorAll('.person-modal');
      var anyOpen = false;
      for (var i = 0; i < modals.length; i++) {
        var isOpen = modals[i].id === current;
        modals[i].classList.toggle('is-open', isOpen);
        modals[i].setAttribute('aria-hidden', isOpen ? 'false' : 'true');
        if (isOpen) anyOpen = true;
      }
      document.body.classList.toggle('has-modal-open', anyOpen);
    }

    document.addEventListener('click', function (e) {
      var tile = e.target.closest ? e.target.closest('.person-tile') : null;
      if (tile) {
        e.preventDefault();
        openModal(tile.getAttribute('href').replace('#', ''), tile);
        return;
      }
      var closer = e.target.closest ? e.target.closest('.person-modal__close, .person-modal__backdrop') : null;
      if (closer) {
        e.preventDefault();
        closeModal();
      }
    });

    document.addEventListener('keydown', function (e) {
      if (e.key === 'Escape' || e.keyCode === 27) closeModal();
    });

    window.addEventListener('hashchange', sync);
    sync();
  })();
</script>
