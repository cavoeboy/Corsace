<template>
    <div class="choice-container">
        <div 
            class="choice box"
            :class="`box--${viewTheme}`"
        >
            <div class="choice__content">
                <a
                    class="choice__img"
                    :href="link"
                    target="_blank"
                >
                    <div
                        class="choice__img"
                        :style="`background-image: url(${imageUrl})`"
                    />
                </a>
    
                <div class="choice__info">
                    <slot />
                </div>
            </div>

            <div
                class="choice__actions"
                :class="{ 
                    'choice__actions--selected': currentNomination,
                    'choice__actions--denied': invalidNomination
                }"
            >
                <div
                    v-if="stage === 'nominating'"
                    class="choice__nominate"
                >
                    {{ currentNomination ? "NOMINATED" : "NOMINATE" }}
                </div>
                <div
                    v-else-if="stage === 'voting'"
                >
                    {{ currentVote ? "VOTED" : "VOTE" }}
                </div>
                <div class="choice__value-container">
                    <div
                        v-if="currentSelected"
                        class="choice__value choice__value--processing"
                    >
                        ...
                    </div>
                    <div
                        v-else-if="invalidNomination"
                        class="choice__value choice__value--denied"
                    >
                        ✕
                    </div>
                    <div
                        v-else
                        class="choice__value"
                        :class="{
                            'choice__value--nominating': stage === 'nominating',
                        }"
                        @click="triggerAction"
                    >
                        <svg
                            v-if="currentNomination"
                            xmlns="http://www.w3.org/2000/svg"
                            width="16"
                            height="16"
                            fill="currentColor"
                            viewBox="0 0 16 16"
                        >
                            <path d="M12.736 3.97a.733.733 0 0 1 1.047 0c.286.289.29.756.01 1.05L7.88 12.01a.733.733 0 0 1-1.065.02L3.217 8.384a.757.757 0 0 1 0-1.06.733.733 0 0 1 1.047 0l3.052 3.093 5.4-6.425a.247.247 0 0 1 .02-.022Z" />
                        </svg>
                        <span v-else-if="stage === 'voting'">
                            {{ currentVote && currentVote.choice || '!' }}
                        </span>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { Vue, Component, Prop } from "vue-property-decorator";
import { namespace, State } from "vuex-class";

import { StageType } from "../../../Interfaces/mca";
import { Vote } from "../../../Interfaces/vote";
import { Nomination } from "../../../Interfaces/nomination";
import { Phase } from "../../../Interfaces/mca";

const mcaAyimModule = namespace("mca-ayim");
const stageModule = namespace("stage");

@Component
export default class BaseChoiceCard extends Vue {

    @Prop({ type: Object, default: () => ({}) }) readonly choice!: Record<string, any>;
    @Prop({ type: String, default: () => "" }) readonly imageUrl!: string;
    @Prop({ type: String, default: () => "" }) readonly link!: string;

    @State viewTheme!: "light" | "dark";

    @mcaAyimModule.Getter phase!: Phase | null;

    @stageModule.State selected!: boolean;
    @stageModule.State stage!: StageType;
    @stageModule.Getter relatedCandidacies!: Vote[] | Nomination[];
    @stageModule.Action createVote;
    @stageModule.Action removeVote;
    @stageModule.Action createNomination;
    @stageModule.Action removeNomination;

    currentSelected = false;

    get currentVote (): Vote | undefined {
        if (this.stage === "nominating") return undefined;

        return (this.relatedCandidacies as Vote[]).find(v => {
            if (this.choice.id && parseInt(this.$route.params.year) < 2021) return v.beatmapset?.ID === this.choice.id;
            else if (this.choice.id && parseInt(this.$route.params.year) >= 2021) return v.beatmap?.ID === this.choice.id;
            else return v.user?.ID === this.choice.corsaceID;
        });
    }

    get currentNomination (): Nomination | undefined {
        if (this.stage === "voting") return undefined;

        return (this.relatedCandidacies as Nomination[]).find(v => {
            if (this.choice.id && parseInt(this.$route.params.year) < 2021) return v.beatmapset?.ID === this.choice.id;
            else if (this.choice.id && parseInt(this.$route.params.year) >= 2021) return v.beatmap?.ID === this.choice.id;
            else return v.user?.ID === this.choice.corsaceID;
        });
    }

    get invalidNomination (): boolean {
        return (this.currentNomination && !this.currentNomination.isValid) || false;
    }

    async triggerAction () {
        if (this.stage === "nominating") {
            await this.nominate();
        } else {
            await this.vote();
        }
    }

    async vote () {
        if (this.selected) return;
        if (!this.phase || this.phase.phase !== this.stage) return;

        this.currentSelected = true;

        if (this.currentVote) {
            await this.removeVote(this.currentVote.ID);
            this.currentSelected = false;
            return;
        }
        
        const id = this.choice.id || this.choice.corsaceID;

        let vote = 1;
        if (this.relatedCandidacies.length) {
            vote = (this.relatedCandidacies as Vote[]).sort((a, b) => b.choice - a.choice)[0].choice + 1;
        }

        await this.createVote({ 
            nomineeId: id,
            vote,
        });

        this.currentSelected = false;
    }

    async nominate () {
        if (this.selected) return;
        if (!this.phase || this.phase.phase !== this.stage) return;

        this.currentSelected = true;

        if (this.currentNomination) {
            await this.removeNomination(this.currentNomination.ID);
            this.currentSelected = false;
            return;
        }
        
        const id = this.choice.id || this.choice.corsaceID;
        await this.createNomination(id);

        this.currentSelected = false;
    }

}
</script>

<style lang="scss">
@import '@s-sass/_variables';
@import '@s-sass/_mixins';
@import '@s-sass/_partials';

.choice {
    &-container {
        flex-grow: 1;
    }

    min-height: 150px;
    padding: 0;

    &__content {
        display: flex;
        flex-direction: column;
        flex: 1;
    }

    &__nominate {
        align-self: center;
        font-size: $font-sm;
        writing-mode: vertical-rl;
        text-orientation: mixed;
        font-weight: bold;
    }

    &__img {
        background-repeat: no-repeat;
        background-size: cover;
        width: auto;
        height: 100%;
    }

    &__info {
        display: flex;
        flex-direction: column;
        justify-content: space-between;

        width: auto;
        height: 100%;
        padding: 5px 10px;
    }

    &__text {
        @extend %text-wrap;
        width: 175px;
        @include breakpoint(mobile) {
            width: 100px;
            font-size: $font-sm;
        }
        @include breakpoint(tablet) {
            width: 250px;
        }
        &--user {
            width: 150px;
            @include breakpoint(mobile) {
                width: 75px;
            }
        }

        &--light {
            color: black;
        }
        &--dark {
            color: white;
        }
        
        &--title {
            color: var(--selected-mode);
        }

        &--subtitle {
            font-size: $font-sm;
            @include breakpoint(mobile) {
                font-size: $font-xsm;
            }
        }

        & div {
            text-overflow: ellipsis;
            overflow: hidden;
            white-space: nowrap;
        }
    }

    &__actions {
        display: flex;
        flex-direction: column;
        justify-content: space-evenly;
        align-items: center;

        background-color: $gray;
        color: white;
        min-width: 60px;
        @include breakpoint(mobile) {
            min-width: 45px;
        }
        @include breakpoint(tablet) {
            min-width: 75px;
        }

        &--selected {
            background-color: var(--selected-mode);
        }

        &--denied {
            background-color: $red;
        }
        
    }

    &__value {
        display: flex;
        justify-content: center;
        align-items: center;

        border-radius: 100%;
        border: 2px solid white;
        width: 50px;
        height: 50px;
        @include breakpoint(mobile) {
            width: 40px;
            height: 40px;
        }
        font-size: $font-xxl;

        & svg {
            width: 80%;
            height: 80%;
        }

        cursor: pointer;

        &:hover {
            background-color: white;
            color: var(--selected-mode);
        }

        &--processing, &--denied {
            cursor: default;
        }

        &--nominating {
            border-radius: 10%;
            width: 35px;
            height: 35px;
        }
    }
}
</style>
