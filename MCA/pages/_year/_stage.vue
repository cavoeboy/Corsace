<template>
    <div class="page">
        <mode-header />
        <div 
            v-if="onTime"
            class="stage-wrapper"
        >
            <stage-page />
        </div>
        <div 
            v-else
            class="wrongPlace"
        >
            <p>
                Ummm... are you in the wrong place?
            </p>
            <a @click="goBack">
                Let's get you to the front page.
            </a>
        </div>
        <notice-modal
            v-if="phase && phase.phase === 'voting' && $route.params.stage === 'voting'"
            :title="$t('mca.main.voting')"
            :text="$t('mca.nom_vote.votingOverlay')"
            :local-key="'voting'"
        />
    </div>
</template>

<script lang="ts">
import { Vue, Component } from "vue-property-decorator";
import { State, namespace } from "vuex-class";

import { MCA, Phase } from "../../../Interfaces/mca";
import { UserMCAInfo } from "../../../Interfaces/user";

import StagePage from "../../components/stage/StagePage.vue";
import NoticeModal from "../../../Assets/components/NoticeModal.vue";
import ModeHeader from "../../../Assets/components/mca-ayim/ModeHeader.vue";

const mcaAyimModule = namespace("mca-ayim");

@Component({
    components: {
        StagePage,
        ModeHeader,
        NoticeModal,
    },
    validate ({ params }): boolean {
        const stageRegex = /^(nominating|nominate|vote|voting)$/i;

        // /2020/nominating
        return /^20\d\d$/.test(params.year) && stageRegex.test(params.stage);
    },
    head () {
        return {
            title: `${this.$route.params.stage} | MCA ${this.$route.params.year ?? (new Date()).getUTCFullYear()}`,
            meta: [
                { hid: "description", name: "description", content: `Mappers' Choice Awards ${this.$route.params.stage} stage is the ${this.$route.params.stage} stage for the osu!-related awards event for mappers for the ${this.$route.params.year ?? (new Date()).getUTCFullYear()} year.` },
                { hid: "og:title", property: "og:title", content: `${this.$route.params.stage} | MCA ${this.$route.params.year ?? (new Date()).getUTCFullYear()}` },
                { hid: "og:type", property: "og:type", content: "website" },
                { hid: "og:url", property: "og:url", content: "https://mca.corsace.io" },
                { hid: "og:description", property: "og:description", content: `Mappers' Choice Awards ${this.$route.params.stage} stage is the ${this.$route.params.stage} stage for the osu!-related awards event for mappers for the ${this.$route.params.year ?? (new Date()).getUTCFullYear()} year.` },
                { hid: "og:site_name", property: "og:site_name", content: "MCA" },
                { hid: "theme-color", name: "theme-color", content: "#fb2475" },
            ],
        };
    },
})
export default class Stage extends Vue {

    @State viewTheme!: "light" | "dark";

    @mcaAyimModule.State selectedMode!: string;
    @mcaAyimModule.State loggedInMCAUser!: UserMCAInfo | null;
    @mcaAyimModule.State mca!: MCA | null;
    @mcaAyimModule.Getter phase!: Phase | null;
    @mcaAyimModule.Mutation toggleGuestDifficultyModal;
    
    mounted () {
        
        if (!this.loggedInMCAUser || !this.loggedInMCAUser.eligibility.some(eligibility => eligibility.year == parseInt(this.$route.params.year)))
            this.$router.push("/" + this.$route.params.year);
    }

    get remainingDays (): string {
        if (this.phase) {
            const date = Math.floor((this.phase.endDate.getTime() - Date.now()) / (1000 * 60 * 60 * 24));
            return date > 9 ? date.toString() : "0" + date;
        }

        return "0";
    }

    get onTime () {
        return this.phase?.phase && (((this.phase.phase === "nominating" || this.phase.phase === "voting") && this.phase.phase === this.$route.params.stage) || (this.mca && this.mca[this.$route.params.stage === "nominating" ? "nomination" : this.$route.params.stage].start <= new Date()));
    }

    goBack () {
        this.$router.push("/" + this.$route.params.year);
    }
}
</script>

<style lang="scss">
@import '@s-sass/_variables';
@import '@s-sass/_mixins';
@import '@s-sass/_partials';

.page {
    padding: 10px 40px;
    @include breakpoint(mobile) {
        padding: 75px 40px;
    }
}

.stage-wrapper {
    width: 100%;
    max-height: 200%;
    padding-top: 10px;

    @include breakpoint(laptop) {
        height: 100%;
    }
}

.remaining-days {
    background-color: white;
    margin-right: auto;
    margin-left: auto;
    border-radius: 0 0 10px 10px;
    padding-left: 0.7%;
    padding-top: 0.5%;
    display: flex;
    align-items: center;
    line-height: calc(1/6);
    overflow: hidden;
    z-index: -100;

    @include breakpoint(laptop) { 
        position: absolute;
        left: 5%;
    }

    @include mode-text-color;

    &__number {
        font-size: 4rem;
        font-weight: bold;
        @include breakpoint(desktop) { 
            font-size: 6rem;
        }
    }

    &__left {
        font-size: 1rem;
        padding: 0 4px;
        letter-spacing: 1px;
        @include breakpoint(desktop) { 
            font-size: 1.5rem;
        }
    }

    &__exclamation {
        font-size: 8rem;
        font-weight: 900;
        transform: rotate(30deg);
        margin-bottom: 10%;
        @include breakpoint(desktop) { 
            font-size: 12rem;
        }
    }
}

.wrongPlace {
    @extend %flex-box;
    flex-direction: column;
    width: 100%;
    justify-content: center;
    align-items: center;
    font-size: 2rem;
}
</style>
