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
          {{ tab }} <!-- Display tab name -->
        </div>
      </div>
    </header>
    <div class="content">
    <div v-if="selectedTab === 'Startseite'">
      <h1>Startseite</h1>
      <p>Willkommen zur Startseite!</p>
        <div class="news-container" v-if="news.length">
          <div v-for="article in news" :key="article.url" class="news-article">
            <h2>{{ article.title }}</h2>                            <!-- Show title -->
            <p>{{ article.description }}</p>                        <!-- Show description -->
            <a :href="article.url" target="_blank">Weiterlesen</a>  <!-- Show link to full article -->
          </div>
        </div>
        <div v-else>
          <p>Keine Nachrichten verfügbar.</p> <!-- Message if no news available -->
        </div>
      </div>
      <div v-if="selectedTab === 'Folge ich'">
        <h1>Folge ich</h1>
        <p>Nachrichten basierend auf deinen gespeicherten Suchbegriffen:</p>
        <div class="news-container" v-if="filteredNews.length">
          <div v-for="article in filteredNews" :key="article.url" class="news-article">
            <h2>{{ article.title }}</h2>                            <!-- Show filtered article title -->
            <p>{{ article.description }}</p>                        <!-- Show filtered article description -->
            <a :href="article.url" target="_blank">Weiterlesen</a>
          </div>
        </div>
        <div v-else>
          <p>Keine Nachrichten verfügbar.</p> <!-- If no filtered news were found -->
        </div>
      </div>
    </div>
    <!-- Restliche Tabs -->
  </div>
</template>

<script>
import axios from 'axios'; // Axios import for news API

export default {
  name: 'App',
  data() {
    return {
      tabs: ['Startseite', 'Für mich', 'Folge ich', 'News Showcase'], // Tab names
      selectedTab: 'Startseite',  // Default selected tab
      news: [],                   // Arry to store news articles 
      searchTerm: '',             // Current search term
      savedSearchTerms: [],       // Array of saved search terms
    };
  },
  computed: {
  filteredNews() {
    return this.news.filter(article => {
      const title = article.title || '';              // Ensure title is not null
      const description = article.description || '';  // Ensure description is not null
      return this.savedSearchTerms.some(term =>
        title.includes(term) || description.includes(term)  // Check if title or description includes any saved term 
      );
    });
  }
  },
  methods: {
    async fetchNews() {
      try {
        const response = await axios.get('https://newsapi.org/v2/top-headlines', {
          params: {
            country: 'de',                              // Country code for Germany
            apiKey: '10f70c5f0807412da77d2a06379a6012'  // My API key
          }
        });
        this.news = response.data.articles;             // Stores fetched articles in news array
      } catch (error) {
        console.error('Fehler beim Abrufen der Nachrichten:', error); // Log error if fetching fails
      }
    },
    async searchNews() {
      if (!this.searchTerm) {
        this.fetchNews();                   // Fetch default news if searchTerm empty
        return;
      }
      try {
        const response = await axios.get('https://newsapi.org/v2/everything', {
          params: {
            q: this.searchTerm,             // Query parameter for search term
            language: 'de',
            apiKey: '10f70c5f0807412da77d2a06379a6012'
          }
        });
        this.news = response.data.articles; // Store fetched articles in news array
      } catch (error) {
        console.error('Fehler beim Suchen der Nachrichten:', error);  // Log error if searching fails
      }
  },
  saveSearchTerm() {
      if (this.searchTerm && !this.savedSearchTerms.includes(this.searchTerm)) {
        this.savedSearchTerms.push(this.searchTerm);  // Add search term to savedSearchTerms array
        this.searchTerm = '';                         // Clear the search term input
      }
    }
  },
  created() {
    this.fetchNews();   // Fetch news when the component is created
  }
};
</script>