<template>
  <div class="app">
    <header class="header container">
      <div class="nav-left">
        <img class="logo" src="../images/logo.svg" alt="Shortly" />
        <nav class="desktop-nav">
          <a href="#">Features</a>
          <a href="#">Pricing</a>
          <a href="#">Resources</a>
        </nav>
      </div>

      <div class="nav-right desktop-nav">
        <a href="#">Login</a>
        <button class="btn btn-pill">Sign Up</button>
      </div>

      <button class="menu-btn" type="button" @click="mobileMenuOpen = !mobileMenuOpen" aria-label="Menu">
        <span></span>
        <span></span>
        <span></span>
      </button>

      <div v-if="mobileMenuOpen" class="mobile-menu">
        <a href="#">Features</a>
        <a href="#">Pricing</a>
        <a href="#">Resources</a>
        <hr />
        <a href="#">Login</a>
        <button class="btn btn-pill btn-mobile">Sign Up</button>
      </div>
    </header>

    <section class="hero container">
      <div class="hero-copy">
        <h1>More than just shorter links</h1>
        <p>Build your brand’s recognition and get detailed insights on how your links are performing.</p>
        <button class="btn btn-pill">Get Started</button>
      </div>

      <div class="hero-art">
        <img src="../images/illustration-working.svg" alt="Working illustration" />
      </div>
    </section>

    <section class="stats-wrap">
      <div class="container">
        <div class="shortener-box">
          <form class="shortener-form" @submit.prevent="shortenUrl">
            <input
              v-model.trim="inputUrl"
              :class="['url-input', { invalid: errorMessage }]"
              type="text"
              placeholder="Shorten a link here..."
              autocomplete="off"
            />
            <button class="btn submit-btn" type="submit">Shorten It!</button>
          </form>
          <p v-if="errorMessage" class="error">{{ errorMessage }}</p>
        </div>

        <ul class="links-list" v-if="links.length">
          <li v-for="link in links" :key="link.code" class="link-item">
            <p class="original-link">{{ link.originalUrl }}</p>
            <div class="right-link">
              <a :href="link.shortUrl" target="_blank" rel="noopener noreferrer">{{ link.shortUrl }}</a>
              <button
                class="btn copy-btn"
                :class="{ copied: copiedCode === link.code }"
                type="button"
                @click="copyLink(link.shortUrl, link.code)"
              >
                {{ copiedCode === link.code ? 'Copied!' : 'Copy' }}
              </button>
            </div>
          </li>
        </ul>

        <section class="advanced">
          <h2>Advanced Statistics</h2>
          <p>
            Track how your links are performing across the web with our advanced statistics dashboard.
          </p>

          <div class="feature-row">
            <article class="feature-card">
              <div class="icon-wrap"><img src="../images/icon-brand-recognition.svg" alt="Brand recognition" /></div>
              <h3>Brand Recognition</h3>
              <p>
                Boost your brand recognition with each click. Generic links don’t mean a thing. Branded links help
                instill confidence in your content.
              </p>
            </article>

            <article class="feature-card raised-1">
              <div class="icon-wrap"><img src="../images/icon-detailed-records.svg" alt="Detailed records" /></div>
              <h3>Detailed Records</h3>
              <p>
                Gain insights into who is clicking your links. Knowing when and where people engage with your content
                helps inform better decisions.
              </p>
            </article>

            <article class="feature-card raised-2">
              <div class="icon-wrap"><img src="../images/icon-fully-customizable.svg" alt="Fully customizable" /></div>
              <h3>Fully Customizable</h3>
              <p>
                Improve brand awareness and content discoverability through customizable links, supercharging audience
                engagement.
              </p>
            </article>
          </div>
        </section>
      </div>
    </section>

    <section class="boost">
      <div class="container boost-inner">
        <h2>Boost your links today</h2>
        <button class="btn btn-pill">Get Started</button>
      </div>
    </section>

    <footer class="footer">
      <div class="container footer-inner">
        <img class="footer-logo" src="../images/logo.svg" alt="Shortly" />

        <div class="footer-col">
          <h4>Features</h4>
          <a href="#">Link Shortening</a>
          <a href="#">Branded Links</a>
          <a href="#">Analytics</a>
        </div>

        <div class="footer-col">
          <h4>Resources</h4>
          <a href="#">Blog</a>
          <a href="#">Developers</a>
          <a href="#">Support</a>
        </div>

        <div class="footer-col">
          <h4>Company</h4>
          <a href="#">About</a>
          <a href="#">Our Team</a>
          <a href="#">Careers</a>
          <a href="#">Contact</a>
        </div>

        <div class="socials">
          <a href="#"><img src="../images/icon-facebook.svg" alt="Facebook" /></a>
          <a href="#"><img src="../images/icon-twitter.svg" alt="Twitter" /></a>
          <a href="#"><img src="../images/icon-pinterest.svg" alt="Pinterest" /></a>
          <a href="#"><img src="../images/icon-instagram.svg" alt="Instagram" /></a>
        </div>
      </div>
    </footer>
  </div>
</template>

<script>
const STORAGE_KEY = 'shortly-links'
const CODE_LENGTH = 6
const CHARS = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'

export default {
  name: 'App',
  data() {
    return {
      inputUrl: '',
      errorMessage: '',
      links: [],
      copiedCode: '',
      mobileMenuOpen: false
    }
  },
  computed: {
    baseShortUrl() {
      return `${window.location.origin}/#/`
    }
  },
  created() {
    this.loadLinks()
    this.resolveHashRedirect()
  },
  methods: {
    normalizeUrl(url) {
      try {
        return new URL(url).toString()
      } catch (error) {
        return ''
      }
    },
    generateCode() {
      let code = ''

      do {
        code = Array.from({ length: CODE_LENGTH }, () => CHARS[Math.floor(Math.random() * CHARS.length)]).join('')
      } while (this.links.some((link) => link.code === code))

      return code
    },
    shortenUrl() {
      this.errorMessage = ''

      const normalized = this.normalizeUrl(this.inputUrl)
      if (!normalized) {
        this.errorMessage = 'Please add a link'
        return
      }

      const existing = this.links.find((link) => link.originalUrl === normalized)
      if (existing) {
        this.inputUrl = ''
        return
      }

      const code = this.generateCode()
      const shortUrl = `${this.baseShortUrl}${code}`
      const newLink = {
        code,
        originalUrl: normalized,
        shortUrl
      }

      this.links.unshift(newLink)
      this.persistLinks()
      this.inputUrl = ''
    },
    loadLinks() {
      const stored = localStorage.getItem(STORAGE_KEY)
      if (!stored) {
        return
      }

      try {
        const parsed = JSON.parse(stored)
        if (Array.isArray(parsed)) {
          this.links = parsed
        }
      } catch (error) {
        this.links = []
      }
    },
    persistLinks() {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(this.links))
    },
    resolveHashRedirect() {
      const hash = window.location.hash || ''
      if (!hash.startsWith('#/')) {
        return
      }

      const code = hash.replace('#/', '').trim()
      if (!code) {
        return
      }

      const match = this.links.find((link) => link.code === code)
      if (match) {
        window.location.replace(match.originalUrl)
      }
    },
    async copyLink(url, code) {
      try {
        await navigator.clipboard.writeText(url)
        this.copiedCode = code
      } catch (error) {
        this.errorMessage = 'Unable to copy to clipboard. Please copy manually.'
      }
    },
    clearAll() {
      this.links = []
      this.copiedCode = ''
      this.errorMessage = ''
      localStorage.removeItem(STORAGE_KEY)
    }
  }
}
</script>

<style>
body {
  margin: 0;
}

@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@500;700&display=swap');

:root {
  --cyan: hsl(180, 66%, 49%);
  --dark-violet: hsl(257, 27%, 26%);
  --red: hsl(0, 87%, 67%);
  --gray: hsl(0, 0%, 75%);
  --grayish-violet: hsl(257, 7%, 63%);
  --very-dark-blue: hsl(255, 11%, 22%);
  --very-dark-violet: hsl(260, 8%, 14%);
  --light-bg: #f0f1f6;
}

* {
  box-sizing: border-box;
}

#app {
  font-family: 'Poppins', sans-serif;
  color: var(--very-dark-blue);
  background: white;
}

.container {
  width: min(1110px, calc(100% - 48px));
  margin: 0 auto;
}

.app {
  overflow-x: hidden;
}

.header {
  position: relative;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 48px;
}

.nav-left {
  display: flex;
  align-items: center;
  gap: 45px;
}

.logo {
  width: 121px;
}

.desktop-nav {
  display: flex;
  align-items: center;
  gap: 30px;
}

.desktop-nav a {
  text-decoration: none;
  color: var(--grayish-violet);
  font-size: 0.9rem;
}

.desktop-nav a:hover {
  color: var(--very-dark-violet);
}

.btn {
  border: none;
  cursor: pointer;
  font-weight: 700;
  font-family: inherit;
}

.btn-pill {
  border-radius: 999px;
  background: var(--cyan);
  color: white;
  padding: 11px 24px;
}

.btn-pill:hover,
.submit-btn:hover,
.copy-btn:hover {
  background: #9ae3e2;
}

.menu-btn {
  display: none;
  background: transparent;
  border: none;
  padding: 0;
  gap: 4px;
  flex-direction: column;
}

.menu-btn span {
  display: block;
  width: 24px;
  height: 3px;
  background: var(--grayish-violet);
}

.mobile-menu {
  display: none;
}

.hero {
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
  padding: 76px 0 136px;
}

.hero-copy h1 {
  font-size: clamp(2.6rem, 6vw, 5rem);
  line-height: 1.1;
  letter-spacing: -1.5px;
  margin: 0;
}

.hero-copy p {
  color: var(--grayish-violet);
  font-size: 1.25rem;
  margin: 8px 0 30px;
  max-width: 540px;
}

.hero-art {
  position: relative;
}

.hero-art img {
  width: 100%;
  min-width: 640px;
  transform: translateX(70px);
}

.stats-wrap {
  background: var(--light-bg);
}

.shortener-box {
  background: var(--dark-violet) url('../images/bg-shorten-desktop.svg') no-repeat right top;
  border-radius: 10px;
  margin-top: -84px;
  padding: 52px 64px;
}

.shortener-form {
  display: grid;
  grid-template-columns: 1fr 188px;
  gap: 24px;
}

.url-input {
  border: 3px solid transparent;
  border-radius: 10px;
  padding: 0 24px;
  height: 64px;
  font-size: 1.2rem;
  font-family: inherit;
}

.url-input::placeholder {
  color: var(--grayish-violet);
}

.url-input.invalid {
  border-color: var(--red);
}

.submit-btn {
  background: var(--cyan);
  border-radius: 10px;
  color: white;
  font-size: 1.25rem;
}

.error {
  margin: 8px 0 0;
  color: var(--red);
  font-size: 0.9rem;
  font-style: italic;
}

.links-list {
  list-style: none;
  padding: 0;
  margin: 24px 0 0;
  display: grid;
  gap: 16px;
}

.link-item {
  background: white;
  border-radius: 8px;
  padding: 16px 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
}

.original-link {
  margin: 0;
  color: var(--very-dark-blue);
  overflow-wrap: anywhere;
}

.right-link {
  display: flex;
  align-items: center;
  gap: 24px;
}

.right-link a {
  color: var(--cyan);
  text-decoration: none;
}

.copy-btn {
  width: 103px;
  height: 40px;
  border-radius: 6px;
  background: var(--cyan);
  color: white;
}

.copy-btn.copied {
  background: var(--dark-violet);
}

.advanced {
  padding: 112px 0 120px;
  text-align: center;
}

.advanced h2 {
  margin: 0;
  font-size: 2.5rem;
}

.advanced > p {
  color: var(--grayish-violet);
  max-width: 520px;
  margin: 16px auto 95px;
}

.feature-row {
  position: relative;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 30px;
  text-align: left;
}

.feature-row::before {
  content: '';
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  height: 8px;
  background: var(--cyan);
  transform: translateY(-50%);
}

.feature-card {
  position: relative;
  z-index: 1;
  background: white;
  padding: 77px 32px 40px;
  border-radius: 8px;
}

.raised-1 {
  margin-top: 44px;
}

.raised-2 {
  margin-top: 88px;
}

.icon-wrap {
  position: absolute;
  top: -44px;
  left: 32px;
  width: 88px;
  height: 88px;
  border-radius: 50%;
  background: var(--dark-violet);
  display: grid;
  place-items: center;
}

.feature-card h3 {
  margin: 0 0 12px;
}

.feature-card p {
  margin: 0;
  color: var(--grayish-violet);
  font-size: 0.95rem;
  line-height: 1.8;
}

.boost {
  background: var(--dark-violet) url('../images/bg-boost-desktop.svg') no-repeat center;
  color: white;
}

.boost-inner {
  text-align: center;
  padding: 58px 0;
}

.boost-inner h2 {
  margin: 0 0 24px;
  font-size: 2.5rem;
}

.footer {
  background: var(--very-dark-violet);
  color: white;
}

.footer-inner {
  display: grid;
  grid-template-columns: 1fr repeat(3, auto) auto;
  gap: 72px;
  padding: 72px 0;
  align-items: start;
}

.footer-logo {
  width: 121px;
  filter: brightness(0) invert(1);
}

.footer-col {
  display: grid;
  gap: 12px;
}

.footer-col h4 {
  margin: 0 0 10px;
}

.footer-col a {
  color: var(--gray);
  text-decoration: none;
  font-size: 0.92rem;
}

.footer-col a:hover {
  color: var(--cyan);
}

.socials {
  display: flex;
  gap: 24px;
}

@media (max-width: 980px) {
  .desktop-nav {
    display: none;
  }

  .menu-btn {
    display: flex;
  }

  .mobile-menu {
    display: grid;
    gap: 24px;
    text-align: center;
    position: absolute;
    top: 96px;
    left: 24px;
    right: 24px;
    background: var(--dark-violet);
    border-radius: 12px;
    padding: 40px 24px;
    z-index: 20;
  }

  .mobile-menu a {
    color: white;
    text-decoration: none;
    font-weight: 700;
  }

  .mobile-menu hr {
    width: 100%;
    border: 0;
    border-top: 1px solid rgba(255, 255, 255, 0.15);
  }

  .btn-mobile {
    width: 100%;
    height: 48px;
  }

  .hero {
    grid-template-columns: 1fr;
    text-align: center;
    padding: 38px 0 168px;
  }

  .hero-art {
    order: -1;
    margin-bottom: 34px;
    width: 100%;
    overflow: hidden;
  }

  .hero-art img {
    width: min(500px, 145%);
    min-width: 0;
    max-width: none;
    transform: translateX(16%);
  }

  .hero-copy p {
    margin: 14px auto 36px;
    font-size: 1.1rem;
  }

  .shortener-box {
    margin-top: -108px;
    background-image: url('../images/bg-shorten-mobile.svg');
    background-size: cover;
    padding: 24px;
  }

  .shortener-form {
    grid-template-columns: 1fr;
    gap: 16px;
  }

  .url-input,
  .submit-btn {
    height: 48px;
    font-size: 1rem;
  }

  .link-item,
  .right-link {
    flex-direction: column;
    align-items: stretch;
    width: 100%;
  }

  .link-item {
    padding: 16px;
  }

  .right-link {
    border-top: 1px solid #edeef2;
    padding-top: 12px;
    gap: 10px;
  }

  .copy-btn {
    width: 100%;
  }

  .advanced {
    padding: 80px 0;
  }

  .advanced h2 {
    font-size: 1.8rem;
  }

  .advanced > p {
    margin-bottom: 90px;
  }

  .feature-row {
    grid-template-columns: 1fr;
    gap: 92px;
    text-align: center;
  }

  .feature-row::before {
    width: 8px;
    height: 100%;
    top: 0;
    bottom: 0;
    left: 50%;
    right: auto;
    transform: translateX(-50%);
  }

  .raised-1,
  .raised-2 {
    margin-top: 0;
  }

  .icon-wrap {
    left: 50%;
    transform: translateX(-50%);
  }

  .boost {
    background-image: url('../images/bg-boost-mobile.svg');
    background-size: cover;
  }

  .boost-inner h2 {
    font-size: 1.75rem;
  }

  .footer-inner {
    grid-template-columns: 1fr;
    justify-items: center;
    text-align: center;
    gap: 40px;
    padding: 54px 0;
  }
}

@media (max-width: 500px) {
  .container {
    width: calc(100% - 48px);
  }

  .hero {
    padding: 30px 0 150px;
  }

  .hero-art img {
    width: 140%;
    transform: translateX(12%);
  }

  .hero-copy h1 {
    font-size: 2.2rem;
    letter-spacing: -1px;
  }

  .hero-copy p {
    font-size: 1rem;
  }
}
</style>
