<template>
  <main class="ugc-page">
    <div class="ugc-layout">
      <div class="logo">
        <div class="axolotl-logo-svg">
          <img
            src="../assets/logoAxolotl.svg"
            alt="Axolotl Face"
            class="logo-svg"
            fetchpriority="high"
          />
        </div>
        <h1 class="logo-text">SMILING<br />AXOLOTL</h1>
      </div>

      <div v-if="state === 'error'" class="ugc-error-layout">
        <div class="ugc-error-content">
          <h2 class="ugc-error-title">{{ $t("ugc.invalid") }}</h2>
          <p class="ugc-error-message">{{ errorMessage }}</p>
        </div>
      </div>

      <div v-else-if="state === 'ready'" class="ugc-countdown-layout">
        <div class="ugc-message">
          <p class="ugc-message-text">{{ $t("ugc.countdownMessage") }}</p>
          <p class="ugc-timer-display">
            {{ secondsRemaining }} <span>{{ $t("ugc.seconds") }}</span>
          </p>
        </div>

        <p class="ugc-countdown-note">{{ $t("ugc.countdownNote") }}</p>
      </div>

      <div v-else class="ugc-followup-layout">
        <div class="ugc-followup-card">
          <h2 class="ugc-followup-title">
            {{ $t("ugc.fallbackHeader") }}
          </h2>
          <p class="ugc-followup-intro">
            {{ $t("ugc.fallback") }}
          </p>
          <ol class="ugc-followup-steps">
            <li>{{ $t("ugc.firstStep") }}</li>
            <li>{{ $t("ugc.secondStep") }}</li>

            <div class="ugc-code-row">
              <span class="ugc-code-value">{{ code }}</span>
              <CustomButton
                :aria-label="$t('ugc.copyCode')"
                variant="secondary"
                extraClass="ugc-copy-btn"
                @click="copyCode"
              >
                {{ $t(copyLabelKey) }}
              </CustomButton>
            </div>
            <li>{{ $t("ugc.thirdStep") }}</li>
          </ol>

          <div class="ugc-actions-bar">
            <a
              class="ugc-open-btn"
              :href="destinationUrl"
              target="_blank"
              rel="noopener noreferrer"
            >
              {{ $t("ugc.openRedeem") }}
            </a>
          </div>
        </div>
      </div>

      <div class="ugc-socials-badge">
        <p class="ugc-socials-label">{{ $t("ugc.socialsLabel") }}</p>
        <div class="ugc-socials-links">
          <a
            aria-label="Twitter"
            href="https://x.com/smiling_axo"
            class="ugc-social-link"
          >
            <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
              <path
                d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-5.214-6.817L4.99 21.75H1.68l7.73-8.835L1.254 2.25H8.08l4.713 6.231zm-1.161 17.52h1.833L7.084 4.126H5.117z"
              />
            </svg>
          </a>
          <a
            aria-label="LinkedIn"
            href="https://www.linkedin.com/company/smiling-axolotl/"
            class="ugc-social-link"
          >
            <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
              <path
                d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433c-1.144 0-2.063-.926-2.063-2.065 0-1.138.92-2.063 2.063-2.063 1.14 0 2.064.925 2.064 2.063 0 1.139-.925 2.065-2.064 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"
              />
            </svg>
          </a>
          <a
            aria-label="Website"
            href="https://smilingaxolotl.com/"
            class="ugc-social-link"
          >
            <svg
              width="20"
              height="20"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="1.8"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <circle cx="12" cy="12" r="9" />
              <path d="M3 12h18" />
              <path d="M12 3a14.5 14.5 0 0 1 0 18" />
              <path d="M12 3a14.5 14.5 0 0 0 0 18" />
            </svg>
          </a>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
import CustomButton from "./CustomButton.vue";

const DESTINATION_ORIGIN = "https://ro.blox.com";
const DESTINATION_PATH = "/Ebh5";
const ROBLOX_PLACE_ID = "78078671877309";

const COUNTDOWN = 10;
const CODE_PATTERN = /^[A-Za-z0-9_-]{3,64}$/;

export default {
  name: "UgcRedirectPage",
  components: {
    CustomButton,
  },
  data() {
    return {
      code: "",
      logoSrc: new URL("../assets/logoAxolotl.svg", import.meta.url).href,
      destinationUrl: "#",
      state: "loading",
      secondsRemaining: 10,
      errorMessage: "",
      copyLabelKey: "ugc.copyCode",
      timerId: null,
    };
  },
  mounted() {
    this.initialize();
  },
  beforeUnmount() {
    if (this.timerId) {
      clearInterval(this.timerId);
    }
  },
  methods: {
    initialize() {
      this.code = this.extractCodeFromPath();

      if (!this.code) {
        this.setError(this.$t("ugc.noCodeProvided"));
        return;
      }

      if (!CODE_PATTERN.test(this.code)) {
        this.setError(this.$t("ugc.unsupportedCodeFormat"));
        return;
      }

      this.destinationUrl = this.buildDestinationUrl(this.code);
      this.secondsRemaining = COUNTDOWN;
      this.state = "ready";
      this.copyLabelKey = "ugc.copyCode";
      this.startCountdown();
    },
    extractCodeFromPath() {
      const parts = window.location.pathname.split("/").filter(Boolean);
      const value = parts.length >= 2 ? decodeURIComponent(parts[1]) : "";
      return value.trim().toUpperCase();
    },
    buildDestinationUrl(code) {
      const url = new URL(DESTINATION_PATH, DESTINATION_ORIGIN);

      const launchData = JSON.stringify({
        code: code,
      });

      const encodedPlaceId = encodeURIComponent(ROBLOX_PLACE_ID);
      const encodedLaunchData = encodeURIComponent(launchData);

      const directToAppLink = `roblox://placeId=${encodedPlaceId}&launchData=${encodedLaunchData}`;
      const webListingLink = `https://www.roblox.com/games/start?placeId=${encodedPlaceId}&launchData=${encodedLaunchData}`;

      url.searchParams.set("af_dp", directToAppLink);
      url.searchParams.set("af_web_dp", webListingLink);

      return url.toString();
    },
    startCountdown() {
      this.timerId = setInterval(() => {
        this.secondsRemaining -= 1;

        if (this.secondsRemaining <= 0) {
          clearInterval(this.timerId);
          this.timerId = null;
          this.openInNewTab();
        }
      }, 1000);
    },
    openInNewTab() {
      const anchor = document.createElement("a");
      anchor.href = this.destinationUrl;
      anchor.target = "_blank";
      anchor.rel = "noopener noreferrer";
      document.body.appendChild(anchor);
      anchor.click();
      document.body.removeChild(anchor);
      this.state = "opened";
    },
    setError(message) {
      this.state = "error";
      this.errorMessage = message;

      if (this.timerId) {
        clearInterval(this.timerId);
        this.timerId = null;
      }
    },
    async copyCode() {
      try {
        await navigator.clipboard.writeText(this.code);
        this.copyLabelKey = "ugc.copied";
      } catch {
        this.copyLabelKey = "ugc.copyFailed";
      }

      setTimeout(() => {
        this.copyLabelKey = "ugc.copyCode";
      }, 1200);
    },
  },
};
</script>

<style scoped>
.ugc-page {
  min-height: 100vh;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: stretch;
  padding: 0;
  background: #2898ff;
  position: relative;
  overflow: hidden;
}

.ugc-layout {
  width: 100%;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  padding: 32px 20px;
}

.logo {
  position: relative;
  pointer-events: none;
  gap: 0.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.axolotl-logo-svg {
  width: 5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  user-select: none;
  pointer-events: none;
}

.logo-svg {
  width: 100%;
  height: 100%;
  object-fit: contain;
  filter: brightness(0) invert(1);
}

.logo-text {
  font-family: "Dela Gothic One", cursive;
  font-size: 1.3rem;
  color: white;
  text-align: center;
  font-weight: 700;
  line-height: 1;
  user-select: none;
  margin: 0;
  letter-spacing: 1px;
}

.ugc-countdown-layout,
.ugc-error-layout {
  width: 100%;
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 20px;
}

.ugc-message {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  flex: 1;
  text-align: center;
}

.ugc-message-text {
  margin: 0;
  font-family: "Dela Gothic One", cursive;
  font-size: 2.8rem;
  font-weight: 700;
  color: #ffffff;
  line-height: 1.15;
  letter-spacing: 0;
  text-transform: uppercase;
  white-space: pre-line;
}

.ugc-timer-display {
  margin: 0;
  font-family: "Dela Gothic One", cursive;
  font-size: 4.2rem;
  font-weight: 900;
  color: #ffc107;
  line-height: 1;
}

.ugc-timer-display span {
  font-size: 1.4rem;
  vertical-align: super;
}

.ugc-countdown-note {
  margin: 0 0 20px;
  color: #e8f0ff;
  font-size: 0.95rem;
  text-align: center;
}

.ugc-followup-layout {
  width: 100%;
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
}

.ugc-followup-card {
  width: 100%;
  max-width: 560px;
  background: #ffffff;
  border: 1px solid rgba(0, 0, 0, 0.06);
  border-radius: 18px;
  padding: 26px;
  color: #1a1a1a;
  box-shadow: 0 14px 36px rgba(0, 0, 0, 0.2);
}

.ugc-followup-title {
  margin: 0 0 10px;
  font-family: "Dela Gothic One", cursive;
  font-size: 1.45rem;
  text-align: center;
}

.ugc-followup-intro {
  margin: 0 0 12px;
  color: #4d4d4d;
  font-size: 1rem;
  text-align: center;
}

.ugc-followup-steps {
  margin: 0 0 16px;
  padding-left: 18px;
  display: grid;
  gap: 6px;
}

.ugc-followup-steps li {
  line-height: 1.4;
}

.ugc-code-row {
  display: flex;
  gap: 10px;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 12px;
}

.ugc-code-value {
  font-family: "Dela Gothic One", cursive;
  font-size: 1.05rem;
  letter-spacing: 1px;
  background: #f4f6f8;
  color: #1a1a1a;
  border-radius: 10px;
  padding: 10px 12px;
  border: 1px solid #e5e8ec;
  flex: 1;
  min-width: 0;
  overflow-wrap: anywhere;
}

.ugc-actions-bar {
  display: flex;
  justify-content: center;
  width: 100%;
  margin-bottom: 0;
  flex-shrink: 0;
}

.ugc-open-btn,
.ugc-copy-btn {
  padding: 12px 18px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 12px;
  font-weight: 700;
  font-family: "Dela Gothic One", cursive;
  font-size: 0.95rem;
  cursor: pointer;
  text-align: center;
  text-decoration: none;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
}

.ugc-copy-btn {
  white-space: nowrap;
}

.ugc-open-btn {
  background: rgba(255, 255, 255, 0.95);
  color: #2898ff;
  border-color: #ffffff;
}

.ugc-open-btn:hover {
  background: #ffffff;
  border-color: #ffffff;
  transform: translateY(-2px);
}

.ugc-copy-btn {
  background: rgba(255, 255, 255, 0.1);
  color: #ffffff;
  border-color: rgba(255, 255, 255, 0.3);
}

.ugc-copy-btn:hover {
  background: rgba(255, 255, 255, 0.2);
  border-color: rgba(255, 255, 255, 0.5);
  transform: translateY(-2px);
}

.ugc-socials-badge {
  width: fit-content;
  background: rgba(255, 255, 255, 0.95);
  padding: 18px 28px;
  padding-left: calc(28px + 18px);
  padding-right: calc(28px + 18px);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 14px;
  flex-shrink: 0;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15);
  animation: fade-in-up 0.6s ease-out;
  position: relative;

  --ugc-cut-mask:
    conic-gradient(from 45deg at left, #0000, #000 1deg 89deg, #0000 90deg)
      left / 51% 39px repeat-y,
    conic-gradient(from 225deg at right, #0000, #000 1deg 89deg, #0000 90deg)
      right / 51% 39px repeat-y;

  -webkit-mask: var(--ugc-cut-mask);
  mask: var(--ugc-cut-mask);
}

.ugc-socials-badge::before {
  content: "";
  position: absolute;
  top: 20px;
  left: -10px;
  width: 0;
  height: 0;
  border-left: 10px solid transparent;
  border-right: 10px solid rgba(255, 255, 255, 0.95);
  border-top: 10px solid transparent;
  border-bottom: 10px solid transparent;
}

.ugc-socials-badge::after {
  content: "";
  position: absolute;
  top: 20px;
  right: -10px;
  width: 0;
  height: 0;
  border-left: 10px solid rgba(255, 255, 255, 0.95);
  border-right: 10px solid transparent;
  border-top: 10px solid transparent;
  border-bottom: 10px solid transparent;
}

.ugc-socials-label {
  font-family: "Dela Gothic One", cursive;
  font-size: 1.02rem;
  font-weight: 700;
  color: #1a1a1a;
  text-transform: uppercase;
  margin: 0;
}

.ugc-socials-links {
  display: flex;
  gap: 18px;
  align-items: center;
  justify-content: center;
}

.ugc-social-link {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  color: #1a1a1a;
  transition: all 0.3s ease;
  text-decoration: none;
}

.ugc-social-link svg {
  width: 24px;
  height: 24px;
}

.ugc-social-link:hover {
  transform: translateY(-2px) scale(1.15);
  color: #2898ff;
}

.ugc-error-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  text-align: center;
  max-width: 520px;
}

.ugc-error-title {
  margin: 0;
  font-family: "Dela Gothic One", cursive;
  font-size: 2.4rem;
  font-weight: 700;
  color: #ffffff;
  text-transform: uppercase;
}

.ugc-error-message {
  margin: 0;
  font-size: 1.1rem;
  color: #e8f0ff;
  line-height: 1.6;
}

@media (max-width: 640px) {
  .ugc-message-text {
    font-size: 1.85rem;
  }

  .ugc-timer-display {
    font-size: 3rem;
  }

  .ugc-timer-display span {
    font-size: 1rem;
  }

  .ugc-actions-bar {
    max-width: 100%;
  }

  .ugc-followup-card {
    max-width: 100%;
    padding: 18px;
  }

  .ugc-copy-btn {
    flex-shrink: 0;
  }

  .ugc-socials-links {
    gap: 14px;
  }

  .ugc-social-link {
    width: 34px;
    height: 34px;
  }

  .ugc-social-link svg {
    width: 22px;
    height: 22px;
  }
}

@media (max-height: 800px) {
  .ugc-layout {
    padding: 20px;
  }
}

@keyframes fade-in-up {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
