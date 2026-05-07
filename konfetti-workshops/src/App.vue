<template>
  <div class="app">
    <nav class="nav">
      <div class="nav-logo">konfetti<span class="dot">.</span></div>
      <div class="nav-links">
        <a href="#">Discover</a>
        <a href="#">For Teams</a>
        <a href="#">Providers</a>
      </div>
      <button class="nav-cta">Book Now</button>
    </nav>

    <section class="hero">
      <div class="hero-tag">12,000+ experiences across Germany & Austria</div>
      <h1 class="hero-title">Find your<br /><em>next adventure.</em></h1>
      <p class="hero-sub">Workshops, experiences, and unforgettable moments — all in one place.</p>

      <div class="search-bar">
        <div class="search-field">
          <span class="search-icon">⌕</span>
          <input v-model="searchQuery" type="text" placeholder="What do you want to learn?" />
        </div>
        <div class="search-field">
          <span class="search-icon">◎</span>
          <input v-model="locationQuery" type="text" placeholder="City or online" />
        </div>
        <button class="search-btn" @click="filterWorkshops">Search</button>
      </div>

      <div class="category-pills">
        <button
          v-for="cat in categories"
          :key="cat"
          class="pill"
          :class="{ active: activeCategory === cat }"
          @click="setCategory(cat)"
        >{{ cat }}</button>
      </div>
    </section>

    <section class="workshops">
      <div class="section-header">
        <h2>{{ activeCategory === 'All' ? 'Popular Workshops' : activeCategory }}</h2>
        <span class="result-count">{{ filteredWorkshops.length }} results</span>
      </div>

      <div class="grid">
        <div
          v-for="workshop in filteredWorkshops"
          :key="workshop.id"
          class="card"
          @click="selectWorkshop(workshop)"
        >
          <div class="card-img" :style="{ background: workshop.color }">
            <span class="card-emoji">{{ workshop.emoji }}</span>
            <div class="card-badge" v-if="workshop.badge">{{ workshop.badge }}</div>
          </div>
          <div class="card-body">
            <div class="card-meta">
              <span class="card-category">{{ workshop.category }}</span>
              <span class="card-rating">★ {{ workshop.rating }}</span>
            </div>
            <h3 class="card-title">{{ workshop.title }}</h3>
            <p class="card-info">{{ workshop.duration }} · {{ workshop.location }}</p>
            <div class="card-footer">
              <span class="card-price">€{{ workshop.price }}</span>
              <button class="card-btn" @click.stop="bookWorkshop(workshop)">Book</button>
            </div>
          </div>
        </div>
      </div>

      <div v-if="filteredWorkshops.length === 0" class="no-results">
        <p>No workshops found. Try a different search.</p>
      </div>
    </section>

    <div class="modal-overlay" v-if="selectedWorkshop" @click.self="closeModal">
      <div class="modal">
        <button class="modal-close" @click="closeModal">✕</button>
        <div class="modal-img" :style="{ background: selectedWorkshop.color }">
          <span class="modal-emoji">{{ selectedWorkshop.emoji }}</span>
        </div>
        <div class="modal-body">
          <span class="modal-category">{{ selectedWorkshop.category }}</span>
          <h2 class="modal-title">{{ selectedWorkshop.title }}</h2>
          <p class="modal-desc">{{ selectedWorkshop.description }}</p>
          <div class="modal-details">
            <div class="detail"><strong>Duration</strong><span>{{ selectedWorkshop.duration }}</span></div>
            <div class="detail"><strong>Location</strong><span>{{ selectedWorkshop.location }}</span></div>
            <div class="detail"><strong>Group size</strong><span>{{ selectedWorkshop.groupSize }}</span></div>
            <div class="detail"><strong>Rating</strong><span>★ {{ selectedWorkshop.rating }} ({{ selectedWorkshop.reviews }} reviews)</span></div>
          </div>
          <div class="modal-footer">
            <span class="modal-price">€{{ selectedWorkshop.price }} <small>per person</small></span>
            <button class="book-btn" @click="confirmBooking">Book Now</button>
          </div>
        </div>
      </div>
    </div>

    <div class="toast" v-if="showToast">✓ Booking confirmed! Check your email.</div>

    <footer class="footer">
      <div class="footer-logo">konfetti<span class="dot">.</span></div>
      <p>Connecting people with unforgettable experiences across Germany & Austria.</p>
      <div class="footer-links">
        <a href="#">About</a>
        <a href="#">Providers</a>
        <a href="#">Blog</a>
        <a href="#">Contact</a>
      </div>
    </footer>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      searchQuery: '',
      locationQuery: '',
      activeCategory: 'All',
      selectedWorkshop: null,
      showToast: false,
      categories: ['All', 'Cooking', 'Art', 'Fitness', 'Music', 'Craft', 'Online'],
      workshops: [
        { id: 1, title: 'Italian Pasta Making', category: 'Cooking', emoji: '🍝', color: 'linear-gradient(135deg, #f5a623 0%, #e8753a 100%)', duration: '3 hours', location: 'Berlin', price: 65, rating: 4.9, reviews: 128, groupSize: '4–12 people', badge: 'Bestseller', description: 'Learn the art of fresh pasta from scratch with a professional Italian chef. You\'ll make tagliatelle, ravioli, and a classic carbonara to take home.' },
        { id: 2, title: 'Watercolor for Beginners', category: 'Art', emoji: '🎨', color: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)', duration: '2.5 hours', location: 'Munich', price: 45, rating: 4.8, reviews: 94, groupSize: '6–10 people', badge: null, description: 'Discover the meditative world of watercolor painting. No experience needed — just bring your curiosity and leave with a finished piece.' },
        { id: 3, title: 'Yoga Flow & Breathwork', category: 'Fitness', emoji: '🧘', color: 'linear-gradient(135deg, #84fab0 0%, #08aeea 100%)', duration: '90 min', location: 'Online', price: 18, rating: 4.7, reviews: 212, groupSize: 'Up to 30', badge: 'Online', description: 'A grounding 90-minute flow combining vinyasa yoga with intentional breathwork. Suitable for all levels. Join from anywhere in the world.' },
        { id: 4, title: 'Sourdough Bread Baking', category: 'Cooking', emoji: '🍞', color: 'linear-gradient(135deg, #f093fb 0%, #f5576c 100%)', duration: '4 hours', location: 'Hamburg', price: 75, rating: 5.0, reviews: 67, groupSize: '4–8 people', badge: '⭐ Top Rated', description: 'Master the ancient craft of sourdough. You\'ll maintain a live starter, shape your loaves, and go home with two freshly baked sourdoughs.' },
        { id: 5, title: 'Electric Guitar for Adults', category: 'Music', emoji: '🎸', color: 'linear-gradient(135deg, #30cfd0 0%, #330867 100%)', duration: '2 hours', location: 'Cologne', price: 55, rating: 4.6, reviews: 43, groupSize: '2–6 people', badge: null, description: 'Always wanted to play guitar? This beginner-friendly session covers chords, basic riffs, and enough to get you playing your first song by the end.' },
        { id: 6, title: 'Candle Making Workshop', category: 'Craft', emoji: '🕯️', color: 'linear-gradient(135deg, #ffecd2 0%, #fcb69f 100%)', duration: '2 hours', location: 'Düsseldorf', price: 38, rating: 4.8, reviews: 156, groupSize: '4–16 people', badge: null, description: 'Create your own hand-poured soy candles with custom scents and colors. Take home 2–3 candles you made yourself. Perfect for date nights and friends.' },
        { id: 7, title: 'Sushi Masterclass', category: 'Cooking', emoji: '🍱', color: 'linear-gradient(135deg, #a1c4fd 0%, #c2e9fb 100%)', duration: '3 hours', location: 'Frankfurt', price: 70, rating: 4.9, reviews: 88, groupSize: '4–10 people', badge: 'New', description: 'Roll your way through nigiri, maki, and temaki with a professional sushi chef. All ingredients and equipment provided — just bring your appetite.' },
        { id: 8, title: 'Life Drawing Class', category: 'Art', emoji: '✏️', color: 'linear-gradient(135deg, #f6d365 0%, #fda085 100%)', duration: '2 hours', location: 'Stuttgart', price: 30, rating: 4.5, reviews: 31, groupSize: '5–15 people', badge: null, description: 'Improve your figure drawing skills in a relaxed, supportive studio environment. All skill levels welcome. Materials provided.' },
      ]
    }
  },
  computed: {
    filteredWorkshops() {
      return this.workshops.filter(w => {
        const matchesCategory = this.activeCategory === 'All' || w.category === this.activeCategory
        const matchesSearch = !this.searchQuery || w.title.toLowerCase().includes(this.searchQuery.toLowerCase())
        const matchesLocation = !this.locationQuery || w.location.toLowerCase().includes(this.locationQuery.toLowerCase())
        return matchesCategory && matchesSearch && matchesLocation
      })
    }
  },
  methods: {
    setCategory(cat) { this.activeCategory = cat },
    filterWorkshops() {},
    selectWorkshop(workshop) { this.selectedWorkshop = workshop },
    closeModal() { this.selectedWorkshop = null },
    bookWorkshop(workshop) { this.selectedWorkshop = workshop },
    confirmBooking() {
      this.selectedWorkshop = null
      this.showToast = true
      setTimeout(() => { this.showToast = false }, 3000)
    }
  }
}
</script>

<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
:root {
  --bg: #faf9f6; --text: #1a1a1a; --muted: #6b6b6b; --accent: #ff4d6d;
  --card-bg: #ffffff; --border: #e8e4df; --radius: 16px;
  --font-display: 'Playfair Display', Georgia, serif;
  --font-body: 'DM Sans', sans-serif;
}
body { font-family: var(--font-body); background: var(--bg); color: var(--text); line-height: 1.6; }
.nav { display: flex; align-items: center; justify-content: space-between; padding: 1.2rem 2.5rem; border-bottom: 1px solid var(--border); background: var(--bg); position: sticky; top: 0; z-index: 100; }
.nav-logo { font-family: var(--font-display); font-size: 1.6rem; font-weight: 700; }
.dot { color: var(--accent); }
.nav-links { display: flex; gap: 2rem; }
.nav-links a { text-decoration: none; color: var(--muted); font-size: 0.9rem; font-weight: 500; transition: color 0.2s; }
.nav-links a:hover { color: var(--text); }
.nav-cta { background: var(--text); color: white; border: none; padding: 0.6rem 1.4rem; border-radius: 100px; font-family: var(--font-body); font-size: 0.85rem; font-weight: 500; cursor: pointer; transition: background 0.2s; }
.nav-cta:hover { background: var(--accent); }
.hero { padding: 5rem 2.5rem 3rem; max-width: 960px; margin: 0 auto; text-align: center; }
.hero-tag { display: inline-block; background: #fff0f3; color: var(--accent); font-size: 0.8rem; font-weight: 500; padding: 0.4rem 1rem; border-radius: 100px; margin-bottom: 1.5rem; }
.hero-title { font-family: var(--font-display); font-size: clamp(2.8rem, 6vw, 5rem); line-height: 1.1; margin-bottom: 1rem; letter-spacing: -1px; }
.hero-title em { font-style: italic; color: var(--accent); }
.hero-sub { color: var(--muted); font-size: 1.1rem; margin-bottom: 2.5rem; font-weight: 300; }
.search-bar { display: flex; background: white; border: 1.5px solid var(--border); border-radius: 100px; padding: 0.4rem 0.4rem 0.4rem 1rem; max-width: 680px; margin: 0 auto 1.5rem; box-shadow: 0 4px 24px rgba(0,0,0,0.06); }
.search-field { display: flex; align-items: center; flex: 1; gap: 0.5rem; padding: 0 1rem; border-right: 1px solid var(--border); }
.search-field:last-of-type { border-right: none; }
.search-icon { color: var(--muted); font-size: 1.1rem; }
.search-field input { border: none; outline: none; font-family: var(--font-body); font-size: 0.9rem; background: transparent; width: 100%; color: var(--text); }
.search-btn { background: var(--accent); color: white; border: none; padding: 0.75rem 1.8rem; border-radius: 100px; font-family: var(--font-body); font-size: 0.9rem; font-weight: 500; cursor: pointer; white-space: nowrap; }
.search-btn:hover { opacity: 0.9; }
.category-pills { display: flex; gap: 0.5rem; justify-content: center; flex-wrap: wrap; }
.pill { background: white; border: 1.5px solid var(--border); padding: 0.45rem 1.1rem; border-radius: 100px; font-family: var(--font-body); font-size: 0.85rem; cursor: pointer; transition: all 0.2s; color: var(--text); }
.pill:hover { border-color: var(--accent); color: var(--accent); }
.pill.active { background: var(--text); color: white; border-color: var(--text); }
.workshops { max-width: 1100px; margin: 0 auto; padding: 3rem 2rem; }
.section-header { display: flex; align-items: baseline; gap: 1rem; margin-bottom: 2rem; }
.section-header h2 { font-family: var(--font-display); font-size: 1.8rem; }
.result-count { color: var(--muted); font-size: 0.9rem; }
.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 1.5rem; }
.card { background: var(--card-bg); border-radius: var(--radius); overflow: hidden; border: 1px solid var(--border); cursor: pointer; transition: transform 0.2s, box-shadow 0.2s; }
.card:hover { transform: translateY(-4px); box-shadow: 0 12px 40px rgba(0,0,0,0.1); }
.card-img { height: 160px; display: flex; align-items: center; justify-content: center; position: relative; }
.card-emoji { font-size: 3rem; }
.card-badge { position: absolute; top: 12px; left: 12px; background: white; font-size: 0.75rem; font-weight: 600; padding: 0.3rem 0.7rem; border-radius: 100px; color: var(--text); }
.card-body { padding: 1.2rem; }
.card-meta { display: flex; justify-content: space-between; margin-bottom: 0.4rem; }
.card-category { font-size: 0.75rem; color: var(--muted); text-transform: uppercase; letter-spacing: 0.5px; font-weight: 500; }
.card-rating { font-size: 0.8rem; color: #f5a623; font-weight: 500; }
.card-title { font-family: var(--font-display); font-size: 1.1rem; margin-bottom: 0.3rem; line-height: 1.3; }
.card-info { font-size: 0.82rem; color: var(--muted); margin-bottom: 1rem; }
.card-footer { display: flex; justify-content: space-between; align-items: center; }
.card-price { font-size: 1.1rem; font-weight: 700; }
.card-btn { background: var(--text); color: white; border: none; padding: 0.5rem 1.1rem; border-radius: 100px; font-family: var(--font-body); font-size: 0.82rem; font-weight: 500; cursor: pointer; }
.card-btn:hover { background: var(--accent); }
.no-results { text-align: center; padding: 3rem; color: var(--muted); }
.modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.5); display: flex; align-items: center; justify-content: center; z-index: 200; padding: 1rem; }
.modal { background: white; border-radius: var(--radius); width: 100%; max-width: 520px; overflow: hidden; position: relative; max-height: 90vh; overflow-y: auto; }
.modal-close { position: absolute; top: 1rem; right: 1rem; background: rgba(255,255,255,0.9); border: none; width: 32px; height: 32px; border-radius: 50%; cursor: pointer; font-size: 0.9rem; z-index: 10; display: flex; align-items: center; justify-content: center; }
.modal-img { height: 200px; display: flex; align-items: center; justify-content: center; }
.modal-emoji { font-size: 4rem; }
.modal-body { padding: 1.5rem; }
.modal-category { font-size: 0.75rem; color: var(--muted); text-transform: uppercase; letter-spacing: 0.5px; }
.modal-title { font-family: var(--font-display); font-size: 1.8rem; margin: 0.5rem 0; }
.modal-desc { color: var(--muted); font-size: 0.9rem; margin-bottom: 1.5rem; line-height: 1.7; }
.modal-details { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; margin-bottom: 1.5rem; padding: 1rem; background: var(--bg); border-radius: 12px; }
.detail { display: flex; flex-direction: column; gap: 0.2rem; }
.detail strong { font-size: 0.75rem; text-transform: uppercase; letter-spacing: 0.5px; color: var(--muted); }
.detail span { font-size: 0.9rem; font-weight: 500; }
.modal-footer { display: flex; justify-content: space-between; align-items: center; }
.modal-price { font-size: 1.5rem; font-weight: 700; }
.modal-price small { font-size: 0.8rem; color: var(--muted); font-weight: 400; }
.book-btn { background: var(--accent); color: white; border: none; padding: 0.85rem 2rem; border-radius: 100px; font-family: var(--font-body); font-size: 0.95rem; font-weight: 600; cursor: pointer; }
.book-btn:hover { opacity: 0.9; }
.toast { position: fixed; bottom: 2rem; left: 50%; transform: translateX(-50%); background: #1a1a1a; color: white; padding: 0.9rem 1.8rem; border-radius: 100px; font-size: 0.9rem; font-weight: 500; z-index: 300; animation: slideUp 0.3s ease; }
@keyframes slideUp { from { opacity: 0; transform: translateX(-50%) translateY(10px); } to { opacity: 1; transform: translateX(-50%) translateY(0); } }
.footer { border-top: 1px solid var(--border); padding: 3rem 2.5rem; text-align: center; color: var(--muted); }
.footer-logo { font-family: var(--font-display); font-size: 1.4rem; font-weight: 700; margin-bottom: 0.5rem; color: var(--text); }
.footer p { font-size: 0.9rem; margin-bottom: 1.5rem; }
.footer-links { display: flex; gap: 2rem; justify-content: center; }
.footer-links a { text-decoration: none; color: var(--muted); font-size: 0.85rem; }
@media (max-width: 600px) {
  .search-bar { flex-direction: column; border-radius: 16px; padding: 1rem; }
  .search-field { border-right: none; border-bottom: 1px solid var(--border); padding: 0.5rem 0; }
  .search-field:last-of-type { border-bottom: none; }
  .search-btn { border-radius: 12px; }
  .nav-links { display: none; }
  .modal-details { grid-template-columns: 1fr; }
}
</style>