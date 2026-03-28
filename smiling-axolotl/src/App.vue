<template>
  <div id="app">
    <UgcRedirectPage v-if="isUgcRoute" />
    <template v-else-if="!isAdminRoute">
      <StickyNavbar />
      <HeroSection />
      <GrowBrandSection />
      <ProjectsSection />
      <StrategicSection />
      <ServicesSection />
      <FooterSection />
      <LanguageToggle />
    </template>
  </div>
</template>

<script>
import StickyNavbar from './components/StickyNavbar.vue';
import HeroSection from './components/HeroSection.vue';
import GrowBrandSection from './components/GrowBrandSection.vue';
import ProjectsSection from './components/ProjectsSection.vue';
import StrategicSection from './components/StrategicSection.vue';
import ServicesSection from './components/ServicesSection.vue';
import FooterSection from './components/FooterSection.vue';
import LanguageToggle from './components/LanguageToggle.vue';
import UgcRedirectPage from './components/UgcRedirectPage.vue';

export default {
  name: 'App',
  components: {
    StickyNavbar,
    HeroSection,
    GrowBrandSection,
    ProjectsSection,
    StrategicSection,
    ServicesSection,
    FooterSection,
    LanguageToggle,
    UgcRedirectPage,
  },
  computed: {
    isUgcRoute() {
      return /^\/ugc(\/|$)/i.test(window.location.pathname);
    },
    isAdminRoute() {
      return /^\/admin(\/|$)/i.test(window.location.pathname);
    },
  },
  methods: {
    initSmoothScrolling() {
      document.querySelectorAll('a[href^="#"]').forEach(anchor => {
        anchor.addEventListener('click', function (e) {
          e.preventDefault();
          const targetId = this.getAttribute('href').substring(1);
          const target = document.getElementById(targetId);
          if (target) {
            const navbarHeight = 80; // Account for navbar height
            const targetPosition = target.offsetTop - (targetId === 'contact' ? 0 : navbarHeight);
            
            window.scrollTo({
              top: targetPosition,
              behavior: 'smooth'
            });
          }
        });
      });
    }
  },
  mounted() {
    if (this.isAdminRoute) {
      if (!/\/admin\/index\.html$/i.test(window.location.pathname)) {
        window.location.replace(`/admin/index.html${window.location.search}${window.location.hash}`);
      }
      return;
    }

    if (!this.isUgcRoute) {
      this.initSmoothScrolling();
    }
  }
}
</script>

<style>
@import url('./style.css');
</style>
