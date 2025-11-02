<template>
  <nav>
    <router-link to="/">Optical Setup Designer</router-link> 
    <router-link to="/about">About</router-link>
    <router-link to="/llm">🤖 Optical AI Assistant</router-link>
  </nav>
  <router-view/>
</template>

<style>
/* Global App Container */
#app {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #f0f0f0;
  background-color: #1a1a1a; 

}
nav {
  padding: 10px 40px; 
  background-color: #282828; 
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.4); 
  display: flex; 
  align-items: center;
  justify-content: flex-start; 
  border-bottom: 2px solid #444; 
}

nav a {
  font-weight: 600; 
  text-decoration: none; 
  color: #ccc; 
  margin-right: 25px; 
  font-size: 1.05rem;
  transition: color 0.3s ease;
  padding: 5px 0; 
}

nav a:hover {
  color: #00bcd4; 
}

/* Active Link State */
nav a.router-link-exact-active {
  color: #00bcd4; /* Teal accent for active link */
  border-bottom: 2px solid #00bcd4; /* Subtle highlight under the active link */
  padding-bottom: 3px;
}
</style>