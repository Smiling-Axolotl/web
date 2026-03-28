<template>
  <section id="projects" class="projects-section">
    <div class="projects-container">
      <h2 class="projects-title">
        <i18n-t keypath="projects.title" tag="span">
          <template #projects>
            <span class="projects-highlight">{{ $t('projects.projects') }}</span>
          </template>
        </i18n-t>
      </h2>
      
      <div class="projects-carousel">
        <button 
          class="carousel-nav carousel-prev" 
          @click="prevSlide"
          :aria-label="$t('projects.previous')"
        >
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M15 18L9 12L15 6" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>

        <div class="carousel-track-container">
          <Swiper
            :modules="modules"
            :slides-per-view="3"
            :space-between="spaceBetween"
            :centered-slides="true"
            :loop="true"
            :autoplay="{
              delay: 1500,
              disableOnInteraction: false,
              pauseOnMouseEnter: true,
            }"
            :breakpoints="breakpoints"
            :speed="600"
            @swiper="onSwiper"
            @slideChange="onSlideChange"
            class="projects-swiper"
          >
            <SwiperSlide 
              v-for="(project, index) in projects" 
              :key="`${index}-${project.title}`"
              class="swiper-slide-custom"
            >
              <div 
                class="project-card"
                @click="openProjectModal(project, index)"
              >
                <div class="project-image-container">
                  <img 
                    :src="project.image" 
                    :alt="project.title"
                    class="project-image"
                  />
                  <div class="project-content">
                    <div class="project-badges">
                      <span 
                        v-for="(badge, badgeIndex) in project.badges" 
                        :key="badgeIndex"
                        class="project-badge"
                        :class="`badge-${badge.toLowerCase()}`"
                      >
                        {{ badge }}
                      </span>
                    </div>
                    <h3 class="project-title">{{ project.title }}</h3>
                  </div>
                </div>
              </div>
            </SwiperSlide>
          </Swiper>
        </div>

        <button 
          class="carousel-nav carousel-next" 
          @click="nextSlide"
          :aria-label="$t('projects.next')"
        >
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M9 18L15 12L9 6" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>
      </div>

      <!--
      <CustomButton
        variant="tertiary"
        class="shimmer discover-btn"
        @click="discoverMore"
        :aria-label="$t('projects.discover')"
      >
        {{ $t('projects.discover') }}
        <svg class="btn-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M5 12H19M19 12L12 5M19 12L12 19" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </CustomButton>
      -->
    </div>

    <!-- Project Modal -->
    <Transition name="modal">
      <div v-if="selectedProject" class="modal-overlay" @click="closeProjectModal">
        <div class="modal-container" @click.stop>
          <div class="modal-card" :style="{ width: modalCardWidth + 'px' }">
            <div class="modal-image-container">
              <img 
                :src="selectedProject.project.image" 
                :alt="selectedProject.project.title"
                class="modal-image"
              />
            </div>
            <div class="modal-details">
              <button class="modal-close" @click="closeProjectModal" :aria-label="$t('projects.close')">
                <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M18 6L6 18M6 6L18 18" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </button>
              
              <div class="modal-badges">
                <span 
                  v-for="(badge, badgeIndex) in selectedProject.project.badges" 
                  :key="badgeIndex"
                  class="project-badge"
                  :class="`badge-${badge.toLowerCase()}`"
                >
                  {{ badge }}
                </span>
              </div>
              
              <h3 class="modal-title">{{ selectedProject.project.title }}</h3>
              
              <p class="modal-description">{{ selectedProject.project.description }}</p>
              
              <CustomButton 
                v-if="selectedProject.project.url"
                variant="tertiary" 
                class="shimmer modal-visit-btn"
                @click="visitProject(selectedProject.project.url)"
                :aria-label="$t('projects.visit')"
              >
                {{ $t('projects.visit') }}
                <svg class="btn-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6M15 3h6v6M10 14L21 3" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </CustomButton>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </section>
</template>

<script>
import { Swiper, SwiperSlide } from 'swiper/vue';
import { Autoplay } from 'swiper/modules';
import 'swiper/css';
import CustomButton from './CustomButton.vue';

// Import all project JSON files dynamically
const projectModules = import.meta.glob('../data/projects/*.json', { eager: true });

export default {
  name: 'ProjectsSection',
  components: {
    CustomButton,
    Swiper,
    SwiperSlide
  },
  data() {
    return {
      selectedProject: null,
      projects: [],
      swiperInstance: null,
      modules: [Autoplay],
      spaceBetween: 32,
      breakpoints: {
        320: {
          slidesPerView: 1,
          spaceBetween: 16,
        },
        480: {
          slidesPerView: 1,
          spaceBetween: 16,
        },
        768: {
          slidesPerView: 2,
          spaceBetween: 24,
        },
        1024: {
          slidesPerView: 3,
          spaceBetween: 32,
        }
      }
    }
  },
  created() {
    // Load projects from JSON files
    this.loadProjects();
  },
  computed: {
    modalCardWidth() {
      const width = window.innerWidth;
      if (width <= 480) return 320;
      if (width <= 768) return 500;
      if (width <= 1024) return 600;
      return 700;
    }
  },
  methods: {
    loadProjects() {
      // Convert imported modules to project array and filter by display field
      this.projects = Object.values(projectModules)
        .map(module => ({
          title: module.default.title,
          image: module.default.image,
          badges: module.default.tags,
          description: module.default.description,
          url: module.default.url,
          display: module.default.display
        }))
        .filter(project => project.display !== false); // Show if display is true or undefined
    },
    onSwiper(swiper) {
      this.swiperInstance = swiper;
    },
    onSlideChange() {
      // Optional: Add any slide change logic here
    },
    openProjectModal(project, index) {
      this.selectedProject = { project, index };
      document.body.style.overflow = 'hidden';
      // Stop autoplay when modal opens
      if (this.swiperInstance && this.swiperInstance.autoplay) {
        this.swiperInstance.autoplay.stop();
      }
    },
    closeProjectModal() {
      this.selectedProject = null;
      document.body.style.overflow = '';
      // Resume autoplay when modal closes
      if (this.swiperInstance && this.swiperInstance.autoplay) {
        this.swiperInstance.autoplay.start();
      }
    },
    visitProject(url) {
      window.open(url, '_blank');
    },
    nextSlide() {
      if (this.swiperInstance) {
        this.swiperInstance.slideNext();
      }
    },
    prevSlide() {
      if (this.swiperInstance) {
        this.swiperInstance.slidePrev();
      }
    },
    discoverMore() {
      window.open('https://www.roblox.com/communities/12277287/Smiling-Axolotl', '_blank');
    }
  }
}
</script>

<style scoped>
.projects-section {
  background: white;
  padding: 12rem 0 8rem;
  position: relative;
  overflow: hidden;
}

.projects-section::before,
.projects-section::after {
  content: '';
  position: absolute;
  top: 100px;
  bottom: 0;
  width: 200px;
  z-index: 5;
  pointer-events: none;
}

.projects-section::before {
  left: 0;
  background: linear-gradient(to right, white 0%, transparent 100%);
}

.projects-section::after {
  right: 0;
  background: linear-gradient(to left, white 0%, transparent 100%);
}

.projects-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 2rem;
  text-align: center;
}

.projects-title {
  font-family: 'Dela Gothic One', cursive;
  font-size: 3rem;
  color: #1a1a1a;
  font-weight: 900;
  text-transform: uppercase;
  margin-bottom: 1.5rem;
}

.projects-highlight {
  color: #2898ff;
}

.projects-carousel {
  position: relative;
  display: flex;
  align-items: center;
  gap: 2rem;
  margin-bottom: 1.5rem;
}

.carousel-track-container {
  flex: 1;
  overflow: hidden;
  padding: 2rem 0;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  max-height: 400px;
}

/* Swiper Styles */
.projects-swiper {
  width: 100%;
  height: auto;
  padding: 2rem 0;
}

.swiper-slide-custom {
  display: flex;
  justify-content: center;
  align-items: center;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.project-card {
  flex-shrink: 0;
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  transform: scale(0.85);
  opacity: 0.6;
  width: 100%;
}

/* Swiper active slide styles */
.swiper-slide-active .project-card {
  transform: scale(1.1);
  opacity: 1;
  z-index: 10;
  cursor: pointer;
}

.project-card:hover {
  opacity: 1;
}

.project-image-container {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 9;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}

.project-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.project-card:hover .project-image {
  transform: scale(1.05);
}

.project-content {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  padding: 1.5rem;
  background: linear-gradient(to bottom, rgba(0, 0, 0, 0.6) 0%, rgba(0, 0, 0, 0) 30%, rgba(0, 0, 0, 0) 70%, rgba(0, 0, 0, 0.9) 100%);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.project-badges {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  align-items: flex-start;
}

.project-badge {
  padding: 0.4rem 0.9rem;
  border-radius: 20px;
  font-family: 'Dela Gothic One', cursive;
  font-size: 0.65rem;
  text-transform: uppercase;
  letter-spacing: 0.8px;
}

/* Badge color variants */
.badge-ownership {
  background: #2898ff;
  color: white;
}

.badge-partnership {
  background: #FAC020;
  color: #1a1a1a;
}

.badge-third-party {
  background: #FF4757;
  color: white;
}

.badge-game-jam {
  background: #9b59b6;
  color: white;
}

.project-title {
  font-family: 'Dela Gothic One', cursive;
  font-size: 1.4rem;
  color: white;
  text-transform: uppercase;
  text-align: left;
  line-height: 1.2;
  text-shadow: 0 0 8px rgba(0, 0, 0, 0.8);
  margin: 0;
  align-self: flex-end;
}

/* Gradient overlays for edge cards - now handled by ::before and ::after on section */
.carousel-gradient {
  display: none;
}

.carousel-nav {
  background: #2898ff;
  border: none;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(40, 152, 255, 0.4);
  flex-shrink: 0;
  z-index: 20;
  position: relative;
}

.carousel-nav:hover {
  transform: scale(1.15);
  box-shadow: 0 6px 20px rgba(40, 152, 255, 0.5);
}

.carousel-nav:active {
  transform: scale(1.05);
}

.carousel-nav svg {
  width: 28px;
  height: 28px;
  color: white;
}

.discover-btn {
  margin-top: 1rem;
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
}

.discover-btn .btn-icon {
  width: 20px;
  height: 20px;
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.85);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 2rem;
}

.modal-container {
  width: 100%;
  max-width: 900px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-card {
  background: white;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.4);
  max-height: 90vh;
  display: flex;
  flex-direction: column;
  animation: float 6s ease-in-out infinite;
}

@keyframes float {
  0%, 100% {
    transform: translateY(0px);
  }
  50% {
    transform: translateY(-10px);
  }
}

.modal-image-container {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 9;
  overflow: hidden;
}

.modal-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.modal-details {
  padding: 2rem;
  position: relative;
  overflow-y: auto;
  max-height: 400px;
}

.modal-close {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: rgba(255, 255, 255, 0.9);
  border: none;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  z-index: 10;
}

.modal-close:hover {
  background: #FF4757;
  transform: scale(1.1);
}

.modal-close svg {
  width: 20px;
  height: 20px;
  color: #1a1a1a;
  transition: color 0.3s ease;
}

.modal-close:hover svg {
  color: white;
}

.modal-badges {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  margin-bottom: 1rem;
}

.modal-title {
  font-family: 'Dela Gothic One', cursive;
  font-size: 1.8rem;
  color: #1a1a1a;
  text-transform: uppercase;
  line-height: 1.2;
  margin-bottom: 1.5rem;
}

.modal-description {
  font-family: 'Poppins', sans-serif;
  font-size: 1rem;
  color: #666;
  line-height: 1.6;
  margin-bottom: 2rem;
}

.modal-visit-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
}

.modal-visit-btn .btn-icon {
  width: 18px;
  height: 18px;
}

/* Modal Transitions */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.4s ease;
}

.modal-enter-active .modal-card,
.modal-leave-active .modal-card {
  transition: transform 0.5s cubic-bezier(0.68, -0.55, 0.27, 1.55), opacity 0.5s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .modal-card {
  transform: scale(0.7) translateY(-10px);
  opacity: 0;
}

.modal-leave-to .modal-card {
  transform: scale(0.7) translateY(-10px);
  opacity: 0;
}

/* Mobile responsive */
@media (max-width: 1024px) {
  .swiper-slide-active .project-card {
    transform: scale(1.08);
  }
}

@media (max-width: 768px) {
  .projects-title {
    font-size: 2rem;
  }

  .projects-section::before,
  .projects-section::after {
    width: 80px;
  }

  .carousel-track-container {
    max-height: 280px;
    padding: 1.5rem 0;
  }

  .project-card {
    transform: scale(0.9);
    opacity: 0.5;
  }

  .swiper-slide-active .project-card {
    transform: scale(1);
    opacity: 1;
  }

  .carousel-nav {
    width: 50px;
    height: 50px;
  }

  .carousel-nav svg {
    width: 24px;
    height: 24px;
  }

  .projects-carousel {
    gap: 1rem;
  }

  .project-title {
    font-size: 1rem;
  }

  .project-badge {
    font-size: 0.6rem;
    padding: 0.3rem 0.7rem;
  }

  .modal-overlay {
    padding: 1rem;
  }

  .modal-details {
    padding: 1.5rem;
    max-height: 300px;
  }

  .modal-title {
    font-size: 1.3rem;
  }

  .modal-description {
    font-size: 0.9rem;
  }
}

@media (max-width: 480px) {
  .projects-container {
    padding: 0 1rem;
  }

  .carousel-track-container {
    max-height: 240px;
    padding: 1rem 0;
  }

  .project-content {
    padding: 1rem;
  }

  .carousel-nav {
    width: 45px;
    height: 45px;
  }

  .modal-details {
    padding: 1rem;
    max-height: 250px;
  }

  .modal-title {
    font-size: 1.1rem;
  }

  .modal-description {
    font-size: 0.85rem;
  }

  .modal-close {
    width: 35px;
    height: 35px;
    top: 0.75rem;
    right: 0.75rem;
  }

  .modal-close svg {
    width: 18px;
    height: 18px;
  }
}
</style>
