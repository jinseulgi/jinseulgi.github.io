[index.html](https://github.com/user-attachments/files/32193464/index.html)
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>진슬기 — Product Design Student</title>
  <meta name="description" content="진슬기 | 국립공주대학교 디자인컨버전스학과. 관심 분야, 수업과 미래 목표를 소개합니다.">
  <meta name="theme-color" content="#f6f5f0">
  <link rel="stylesheet" href="./styles.css">
  <script>
    // ✏️ 개인 정보 수정: 아래 따옴표 안의 글자만 바꾸세요.
    // 소개는 한 문장, 각 정보 블록은 2~3줄 분량을 권장합니다.
    const profile = {
      name: '진슬기',
      initials: 'SJ',
      affiliation: '국립공주대학교 디자인컨버전스학과',
      introduction: '사람과 기술 사이, 더 나은 방향을 디자인합니다.',
      interest: '사용자 경험과 새로운 기술이 만나는 지점을 탐구합니다.',
      className: '미래첨단기술의 이해 I',
      classDescription: '미래 기술을 이해하고, 디자인의 관점으로 일상의 가능성을 발견합니다.',
      future: '복잡한 기술을 누구나 쉽게 경험할 수 있도록 만드는 디자이너.',
      note: '새로운 가능성을 향해, 한 걸음씩.',
      coordinates: '00° 00′ N / 000° 00′ E', // 실제 위치가 아닌 장식용 좌표입니다.
      email: 'gixeulln@smail.kongju.ac.kr',
      github: 'https://github.com/jinseulgi/jinseulgi.github.io',
      githubLabel: 'jinseulgi.github.io'
    };
    // 관심 분야는 최대 세 개까지 수정할 수 있습니다.
    const interestAreas = [
      { title: '제품디자인', description: '형태와 기능을 연결해 일상에 필요한 경험을 만듭니다.' },
      { title: '사용자 경험', description: '사람의 행동과 맥락을 이해하고 사용하기 쉬운 흐름을 탐구합니다.' },
      { title: 'AI', description: '새로운 기술이 디자인의 과정과 경험을 어떻게 바꾸는지 살펴봅니다.' }
    ];
    document.addEventListener('DOMContentLoaded', () => {
      document.querySelectorAll('[data-profile]').forEach(element => {
        element.textContent = profile[element.dataset.profile];
      });
      document.querySelectorAll('[data-contact="email"]').forEach(a => a.href = 'mailto:' + profile.email);
      document.querySelectorAll('[data-contact="github"]').forEach(a => a.href = profile.github);
      document.title = profile.name + ' — Product Design Student';
      document.querySelector('meta[name="description"]').content = profile.name + ' | ' + profile.affiliation + '. ' + profile.introduction;
    });

    document.addEventListener('DOMContentLoaded', () => {
      const reduced = matchMedia('(prefers-reduced-motion: reduce)');
      const desktop = matchMedia('(hover: hover) and (pointer: fine)');
      const root = document.documentElement;
      const radar = document.querySelector('.radar');
      const sweep = document.querySelector('.radar-sweep');
      const cursor = document.querySelector('.ship-cursor');
      const ship = cursor.querySelector('svg');
      const marker = document.querySelector('.cursor-detection');
      const blocks = [...document.querySelectorAll('.information article')];
      const dots = [...document.querySelectorAll('.radar-contact')];
      const bearings = [45, 180, 285];
      const litAt = [-Infinity, -Infinity, -Infinity];
      let x = 0, y = 0, heading = 0, shipAngle = 0, angle = 0, target = 0;
      let present = false, lastMove = -Infinity, lastFrame = 0, frame = 0, active = -1;
      const difference = (a, b) => ((a - b + 180) % 360 + 360) % 360 - 180;
      function detect(index) {
        if (active === index) return;
        blocks.forEach((block, i) => block.classList.toggle('is-detected', i === index));
        if (index >= 0) litAt[index] = performance.now();
        active = index;
      }
      function leave() {
        present = false;
        lastMove = -Infinity;
        root.classList.remove('ship-visible');
        cursor.classList.remove('is-visible', 'is-clickable');
        marker.classList.remove('is-visible');
        detect(-1);
      }
      function draw(now) {
        const dt = Math.min(now - (lastFrame || now), 50);
        lastFrame = now;
        const previous = angle;
        // 마지막 이동 후 700ms 동안 추적을 마무리한 다음 10초 주기로 회전합니다.
        if (present && now - lastMove < 700) angle += difference(target, angle) * (1 - Math.exp(-dt / 120));
        else angle += dt * .036;
        sweep.style.transform = `rotate(${angle}deg)`;
        if (present) {
          shipAngle += difference(heading, shipAngle) * (1 - Math.exp(-dt / 55));
          ship.style.transform = `rotate(${shipAngle}deg)`;
        }
        const step = angle - previous;
        dots.forEach((dot, i) => {
          const offset = difference(bearings[i], previous);
          if ((step >= 0 && offset >= 0 && offset <= step) || (step < 0 && offset <= 0 && offset >= step)) litAt[i] = now;
          dot.style.opacity = .3 + .7 * Math.exp(-(now - litAt[i]) / 330);
        });
        frame = requestAnimationFrame(draw);
      }
      function configure() {
        cancelAnimationFrame(frame);
        leave();
        root.classList.toggle('mouse-radar', desktop.matches && !reduced.matches);
        root.classList.add('radar-controlled');
        angle = 0; shipAngle = 0; heading = 0; lastFrame = 0;
        sweep.style.transform = 'rotate(0deg)';
        ship.style.transform = 'rotate(0deg)';
        dots.forEach((dot, i) => { dot.style.opacity = '.3'; litAt[i] = -Infinity; });
        if (!reduced.matches && !document.hidden) frame = requestAnimationFrame(draw);
      }
      document.addEventListener('pointermove', event => {
        if (!desktop.matches || event.pointerType !== 'mouse') { leave(); return; }
        const dx = event.clientX - x, dy = event.clientY - y;
        if (present && Math.hypot(dx, dy) > .5) heading = Math.atan2(dx, -dy) * 180 / Math.PI;
        x = event.clientX; y = event.clientY;
        present = true;
        lastMove = performance.now();
        // 위치는 즉시 반영하고 선체 회전에만 짧은 관성을 적용합니다.
        cursor.style.transform = `translate(${x}px, ${y}px)`;
        cursor.classList.add('is-visible');
        root.classList.add('ship-visible');
        cursor.classList.toggle('is-clickable', !!event.target.closest('a, button, input, select, textarea, [role="button"]'));
        if (reduced.matches) return;
        const matrix = radar.getScreenCTM();
        if (!matrix) return;
        const point = new DOMPoint(x, y).matrixTransform(matrix.inverse());
        if (Math.hypot(point.x - 250, point.y - 250) > 3) target = Math.atan2(point.x - 250, 250 - point.y) * 180 / Math.PI;
        marker.setAttribute('cx', point.x);
        marker.setAttribute('cy', point.y);
        marker.classList.toggle('is-visible', Math.hypot(point.x - 250, point.y - 250) <= 172);
        let nearest = -1, distance = 28;
        blocks.forEach((block, i) => {
          const box = block.getBoundingClientRect();
          const gap = Math.hypot(Math.max(box.left - x, 0, x - box.right), Math.max(box.top - y, 0, y - box.bottom));
          if (gap < distance) { distance = gap; nearest = i; }
        });
        detect(nearest);
      }, { passive: true });
      document.documentElement.addEventListener('pointerleave', leave);
      document.addEventListener('pointerdown', event => { if (event.pointerType !== 'mouse') leave(); }, { passive: true });
      window.addEventListener('blur', leave);
      window.addEventListener('scroll', leave, { passive: true });
      window.addEventListener('resize', leave);
      document.addEventListener('visibilitychange', configure);
      reduced.addEventListener('change', configure);
      desktop.addEventListener('change', configure);
      configure();
    });

    document.addEventListener('DOMContentLoaded', () => {
      document.getElementById('current-year').textContent = new Date().getFullYear();
      const field = document.getElementById('interest-areas');
      interestAreas.slice(0, 3).forEach((item, i) => {
        const entry = document.createElement('div');
        entry.className = 'interest-entry';
        const button = document.createElement('button');
        button.type = 'button';
        button.className = 'interest-orbit';
        button.textContent = item.title;
        button.setAttribute('aria-expanded', 'false');
        button.setAttribute('aria-controls', 'interest-description-' + i);
        const description = document.createElement('p');
        description.id = 'interest-description-' + i;
        description.textContent = item.description;
        const open = value => { entry.classList.toggle('is-open', value); button.setAttribute('aria-expanded', String(value)); };
        entry.addEventListener('pointerenter', e => { if (e.pointerType === 'mouse') open(true); });
        entry.addEventListener('pointerleave', () => { if (!entry.contains(document.activeElement)) open(false); });
        button.addEventListener('focus', () => open(true));
        button.addEventListener('blur', () => open(false));
        button.addEventListener('click', e => { if (e.detail !== 0 && e.pointerType === 'mouse') open(true); else open(!entry.classList.contains('is-open')); });
        entry.append(button, description); field.append(entry);
      });
      const sections = [...document.querySelectorAll('.journey > section')];
      const links = [...document.querySelectorAll('.nav-links a')];
      const markers = [...document.querySelectorAll('.scroll-route i')];
      const reduced = matchMedia('(prefers-reduced-motion: reduce)');
      const observer = new IntersectionObserver(entries => entries.forEach(entry => {
        if (entry.isIntersecting) {
          entry.target.classList.add('is-revealed');
          observer.unobserve(entry.target);
        }
      }), { threshold: .15 });
      document.querySelectorAll('.reveal').forEach(element => observer.observe(element));
      document.documentElement.classList.add('scroll-enhanced');
      function updatePosition() {
        let current = 0;
        sections.forEach((section, i) => { if (section.getBoundingClientRect().top <= innerHeight * .4) current = i; });
        if (innerHeight + scrollY >= document.documentElement.scrollHeight - 3) current = sections.length - 1;
        links.forEach((link, i) => { if (i === current) link.setAttribute('aria-current', 'location'); else link.removeAttribute('aria-current'); });
        markers.forEach((marker, i) => marker.classList.toggle('active', i === current));
        const total = document.documentElement.scrollHeight - innerHeight;
        document.querySelector('.scroll-fill').style.transform = `scaleY(${total > 0 ? Math.min(1, scrollY / total) : 0})`;
      }
      let pending = false;
      window.addEventListener('scroll', () => { if (!pending) { pending = true; requestAnimationFrame(() => { updatePosition(); pending = false; }); } }, { passive: true });
      window.addEventListener('resize', updatePosition);
      reduced.addEventListener('change', () => { if (reduced.matches) document.querySelectorAll('.reveal').forEach(e => e.classList.add('is-revealed')); });
      updatePosition();
    });
  </script>
</head>
<body>
  <div class="cursor-layer" aria-hidden="true"><span class="ship-cursor"><svg viewBox="0 0 26 26" width="26" height="26" aria-hidden="true" focusable="false"><path d="M13 2 19 10 18 23H8L7 10Z" fill="white" stroke="#171e30" stroke-width="1.25" stroke-linejoin="round"/><path d="M11 13h4v4h-4z" fill="#244edb"/></svg></span></div>
  <nav class="top-nav" aria-label="주요 섹션">
    <div class="nav-links"><a href="#about" aria-current="location">ABOUT</a><a href="#interest">INTEREST</a><a href="#future">FUTURE</a><a href="#contact">CONTACT</a></div>
  </nav>
  <main class="journey">
  <section class="page" id="about" aria-labelledby="name">
    <header class="header">
      <div class="identity"><span class="initials" data-profile="initials">SJ</span><span class="eyebrow">PRODUCT DESIGN STUDENT</span></div>
      <p class="course-name" data-profile="className">미래첨단기술의 이해 I</p>
    </header>
    <div class="about-layout">
      <section class="introduction" aria-labelledby="name">
        <div class="intro-copy">
          <p class="eyebrow section-label"><span aria-hidden="true">01 /</span> INTRODUCTION · CURRENT POSITION</p>
          <h1 id="name" data-profile="name">진슬기</h1>
          <p class="affiliation" data-profile="affiliation">국립공주대학교 디자인컨버전스학과</p>
          <p class="statement" data-profile="introduction">사람과 기술 사이, 더 나은 방향을 디자인합니다.</p>
        </div>
        <footer class="intro-footer">
          <p data-profile="note">새로운 가능성을 향해, 한 걸음씩.</p>
          <p class="coordinates" data-profile="coordinates" aria-hidden="true">00° 00′ N / 000° 00′ E</p>
          <div class="contacts"><a data-contact="email" href="mailto:gixeulln@smail.kongju.ac.kr">Email ↗</a><a data-contact="github" data-profile="githubLabel" href="https://github.com/jinseulgi/jinseulgi.github.io" target="_blank" rel="noopener noreferrer">GitHub ↗</a></div>
        </footer>
      </section>
      <div class="right-column">
        <div class="graphic" aria-hidden="true">
          <svg class="radar" viewBox="0 0 500 500" fill="none" xmlns="http://www.w3.org/2000/svg" focusable="false" aria-hidden="true">
            <g class="radar-grid">
              <circle cx="250" cy="250" r="60"/>
              <circle cx="250" cy="250" r="120"/>
              <circle cx="250" cy="250" r="180"/>
              <path d="M250 70V430M70 250H430"/>
              <path d="M250.00 66.00L250.00 59.00M342.00 90.65L345.50 84.59M409.35 158.00L415.41 154.50M434.00 250.00L441.00 250.00M409.35 342.00L415.41 345.50M342.00 409.35L345.50 415.41M250.00 434.00L250.00 441.00M158.00 409.35L154.50 415.41M90.65 342.00L84.59 345.50M66.00 250.00L59.00 250.00M90.65 158.00L84.59 154.50M158.00 90.65L154.50 84.59"/>
            </g>
            <g class="directions" fill="currentColor" stroke="none">
              <text x="250" y="42">N</text><text x="463" y="254">E</text>
              <text x="250" y="466">S</text><text x="37" y="254">W</text>
            </g>
            <!-- 마우스 방향을 따라가며, 터치 기기에서는 천천히 자동 회전합니다. -->
            <g class="radar-sweep"><path class="sweep-sector" d="M250 250L212.576 73.933A180 180 0 0 1 250 70Z"/><path d="M250 250V70"/></g>
            <circle cx="250" cy="250" r="2.5" fill="#171e30"/>
            <circle class="cursor-detection" cx="250" cy="250" r="8"/>
            <!-- 북쪽 기준 45°, 180°, 285°: 주사선 도착 시점과 점의 강조를 맞춥니다. -->
            <g class="radar-contacts">
              <circle class="radar-contact contact-one" cx="334.853" cy="165.147" r="3.5"/>
              <circle class="radar-contact contact-two" cx="250" cy="400" r="3.5"/>
              <circle class="radar-contact contact-three" cx="95.452" cy="208.589" r="3.5"/>
            </g>
          </svg>
        </div>
        <section class="information" aria-label="관심 분야, 수업 소개와 미래 목표">
          <article>
            <svg viewBox="0 0 24 24" aria-hidden="true" focusable="false"><circle cx="10" cy="10" r="6"/><path d="m15 15 6 6M10 7v6M7 10h6"/></svg>
            <h2>INTEREST</h2><p data-profile="interest">사용자 경험과 새로운 기술이 만나는 지점을 탐구합니다.</p>
          </article>
          <article>
            <svg viewBox="0 0 24 24" aria-hidden="true" focusable="false"><path d="M12 5v16M12 5C9 2 5 3 2 3v15c4 0 7 0 10 3 3-3 6-3 10-3V3c-3 0-7-1-10 2Z"/></svg>
            <h2>CLASS</h2><p data-profile="classDescription">미래 기술을 이해하고, 디자인의 관점으로 일상의 가능성을 발견합니다.</p>
          </article>
          <article>
            <svg viewBox="0 0 24 24" aria-hidden="true" focusable="false"><path d="M4 20 20 4M7 4h13v13"/></svg>
            <h2>FUTURE</h2><p data-profile="future">복잡한 기술을 누구나 쉽게 경험할 수 있도록 만드는 디자이너.</p>
          </article>
        </section>
      </div>
    </div>
  </section>
  <section id="interest" class="journey-section" aria-labelledby="interest-title">
    <div class="section-content reveal">
      <p class="section-label eyebrow">02 / INTEREST</p>
      <h2 id="interest-title" class="chapter-title">관심의 영역을 탐지합니다.</h2>
      <div class="interest-field" id="interest-areas"></div>
    </div>
  </section>
  <section id="future" class="journey-section" aria-labelledby="future-title">
    <div class="section-content reveal">
      <p class="section-label eyebrow">04 / FUTURE <span class="small-note">NEXT DESTINATION</span></p>
      <h2 id="future-title" class="future-statement" data-profile="future"></h2>
      <svg class="destination-route" viewBox="0 0 1000 160" fill="none" aria-hidden="true" focusable="false"><path d="M10 140H270L410 95H650L800 25H985" stroke="#cbd0d5" stroke-width="1"/><circle cx="10" cy="140" r="3" fill="#171e30"/><circle cx="985" cy="25" r="5" fill="#244edb"/></svg>
    </div>
  </section>
  <section id="contact" class="journey-section contact-section" aria-labelledby="contact-title">
    <div class="section-content reveal">
      <p class="section-label eyebrow">03 / CONTACT</p>
      <h2 id="contact-title" class="chapter-title">연락처</h2>
      <div class="contact-addresses"><a data-contact="email" data-profile="email" href="mailto:gixeulln@smail.kongju.ac.kr"></a><a data-contact="github" data-profile="githubLabel" href="https://github.com/jinseulgi/jinseulgi.github.io" target="_blank" rel="noopener noreferrer"></a></div>
      <footer class="arrival"><span>© <span id="current-year"></span> <span data-profile="name"></span></span><span>ARRIVED</span></footer>
    </div>
  </section>
  </main>
  <div class="scroll-route" aria-hidden="true"><span class="scroll-fill"></span><i class="active"></i><i></i><i></i><i></i></div>
</body>
</html>
