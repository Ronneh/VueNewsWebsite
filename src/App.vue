<template>
  <div id="app">
    <header id="header">
      <div class="search-bar">
        <input type="text" placeholder="Suchen..." v-model="searchTerm" />
        <button class="search-button" @click="searchNews">Suchen</button>
        <button class="save-button" @click="saveSearchTerm">Begriff speichern</button>
      </div>
      <div class="tabs">
        <div
          v-for="tab in tabs"
          :key="tab"
          :class="['tab', { active: selectedTab === tab }]"
          @click="selectedTab = tab"
        >
          {{ tab }}
        </div>
      </div>
    </header>
    <div class="content">
    <div v-if="selectedTab === 'Startseite'">
      <h1>Startseite</h1>
      <p>Willkommen zur Startseite!</p>
        <div class="news-container" v-if="news.length">
          <div v-for="article in news" :key="article.url" class="news-article">
            <h2>{{ article.title }}</h2>
            <p>{{ article.description }}</p>
            <a :href="article.url" target="_blank">Weiterlesen</a>
          </div>
        </div>
        <div v-else>
          <p>Keine Nachrichten verfügbar.</p>
        </div>
      </div>
      <div v-if="selectedTab === 'Folge ich'">
        <h1>Folge ich</h1>
        <p>Nachrichten basierend auf deinen gespeicherten Suchbegriffen:</p>
        <div class="news-container" v-if="filteredNews.length">
          <div v-for="article in filteredNews" :key="article.url" class="news-article">
            <h2>{{ article.title }}</h2>
            <p>{{ article.description }}</p>
            <a :href="article.url" target="_blank">Weiterlesen</a>
          </div>
        </div>
        <div v-else>
          <p>Keine Nachrichten verfügbar.</p>
        </div>
      </div>
    </div>
    <!-- Restliche Tabs hier -->
  </div>
</template>

<script>
import axios from 'axios'; // Axios import for news API

export default {
  name: 'App',
  data() {
    return {
      tabs: ['Startseite', 'Für mich', 'Folge ich', 'News Showcase'],
      selectedTab: 'Startseite',
      news: [],
      searchTerm: '',
      savedSearchTerms: [], // Array of saved search terms
    };
  },
  computed: {
  filteredNews() {
    return this.news.filter(article => {
      const title = article.title || '';
      const description = article.description || '';
      return this.savedSearchTerms.some(term =>
        title.includes(term) || description.includes(term)
      );
    });
  }
  },
  methods: {
    async fetchNews() {
      try {
        const response = await axios.get('https://newsapi.org/v2/top-headlines', {
          params: {
            country: 'de',
            apiKey: '10f70c5f0807412da77d2a06379a6012'
          }
        });
        this.news = response.data.articles;
      } catch (error) {
        console.error('Fehler beim Abrufen der Nachrichten:', error);
      }
    },
    async searchNews() {
      if (!this.searchTerm) {
        this.fetchNews();
        return;
      }
      try {
        const response = await axios.get('https://newsapi.org/v2/everything', {
          params: {
            q: this.searchTerm,
            language: 'de',
            apiKey: '10f70c5f0807412da77d2a06379a6012'
          }
        });
        this.news = response.data.articles;
      } catch (error) {
        console.error('Fehler beim Suchen der Nachrichten:', error);
      }
  },
  saveSearchTerm() {
      if (this.searchTerm && !this.savedSearchTerms.includes(this.searchTerm)) {
        this.savedSearchTerms.push(this.searchTerm);
        this.searchTerm = '';       // Clear the search term input
      }
    }
  },
  created() {
    this.fetchNews();
  }
};
</script>