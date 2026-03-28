<template>
  <section class="strategic-section">
    <div class="strategic-container">
      <div class="strategic-partners-wrapper">
        <div class="strategic-partners-card">
          <div class="card-background"></div>
          <div class="card-content">
            <h3 class="strategic-title">{{ $t('partners.strategic.title') }}</h3>
            <div class="strategic-partners-logos">
              <Tooltip text="Astral Labs" position="bottom">
                <img :src="astralLabsLogo" alt="Astral Labs" class="partner-logo" />
              </Tooltip>
              <Tooltip text="Creating Madness" position="bottom">
                <img :src="creatingMadnessLogo" alt="Creating Madness" class="partner-logo" />
              </Tooltip>
              <Tooltip text="Konect Network" position="bottom">
                <img :src="konectNetworkLogo" alt="Konect Network" class="partner-logo" />
              </Tooltip>
            </div>
          </div>
        </div>
      </div>
      
      <div class="experience-wrapper">
        <div class="experience-card">
          <div class="card-background"></div>
          <div class="card-content">
            <h3 class="experience-title">{{ $t('partners.experience.title') }}</h3>
            <div class="experience-logos">
              <Tooltip text="Netflix Next World" position="bottom">
                <img :src="netflixNextWorldLogo" alt="Netflix Next World" class="partner-logo no-filter-logo" />
              </Tooltip>
              <Tooltip text="Baobab Studios" position="bottom">
                <img :src="baobabLogo" alt="Baobab Studios" class="partner-logo no-filter-logo" />
              </Tooltip>
              <Tooltip text="Sparko Studio" position="bottom">
                <img :src="sparkoStudioLogo" alt="Sparko Studio" class="partner-logo no-filter-logo" />
              </Tooltip>
              <Tooltip text="Second Chance Productions LLC" position="bottom">
                <img :src="secondChanceLogo" alt="Second Chance Productions LLC" class="partner-logo no-filter-logo" />
              </Tooltip>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import Tooltip from './Tooltip.vue';
import CreatingMadnessLogo from '../assets/partners/CreatingMadness.svg';
import AstralLabsLogo from '../assets/partners/AstralLabs.svg';
import KonectNetworkLogo from '../assets/partners/KonectNetwork.svg';
import BaobabLogo from '../assets/experience/Baobab.svg';
import SecondChanceLogo from '../assets/experience/SecondChanceProd.svg';
import NetflixNextWorldLogo from '../assets/experience/NetflixNextWorld.svg';
import SparkoStudio from '../assets/experience/SparkoStudio.svg';

export default {
  name: 'StrategicSection',
  components: {
    Tooltip
  },
  data() {
    return {
      creatingMadnessLogo: CreatingMadnessLogo,
      astralLabsLogo: AstralLabsLogo,
      konectNetworkLogo: KonectNetworkLogo,
      baobabLogo: BaobabLogo,
      secondChanceLogo: SecondChanceLogo,
      netflixNextWorldLogo: NetflixNextWorldLogo,
      sparkoStudioLogo: SparkoStudio,
    }
  }
}
</script>

<style scoped>
</style>
