<template>
	<div
		class="stepper-box"
		:class="$store.getters.activeStepIndex === $store.getters.steps.length - 1 ? 'laststep' : ''"
	>
		<div class="top">
			<div class="steps-wrapper">
				<round-slider
					v-model="$store.getters.activeStepIndex + 1"
					radius="35"
					width="8"
					rangeColor="#d34ffe"
					readOnly="true"
					handleSize="0"
					min="0"
					:max="$store.getters.steps.length"
				></round-slider>

				<template v-if="topButtons">
					<div v-if="currentStep.index > 0" class="stepper-button-top previous" @click="backStep()">
						<i class="material-icons">keyboard_arrow_left</i>
					</div>
				</template>
				<h3 class="number-of-steps">of {{ $store.getters.steps.length }}</h3>
				<div ref="stepsParent">
					<template v-for="(step, index) in steps">
						<div
							ref="stepsChild"
							:class="['step', isStepActive(index, step)]"
							:key="index"
							:style="{width: `${100 / steps.length}%`}"
						>
							<div class="circle">
								<i class="material-icons md-18">
									{{ step.completed ? 'done' : step.icon }}
								</i>
							</div>
							<div class="step-title">
								<h4>{{ step.title }}</h4>
								<h5 class="step-subtitle">{{ step.subtitle }}</h5>
							</div>
						</div>
					</template>
				</div>

				<div
					v-if="topButtons"
					:class="['stepper-button-top next', !canContinue ? 'deactivated' : '']"
					@click="nextStep()"
				>
					<i class="material-icons">keyboard_arrow_right</i>
				</div>
			</div>
		</div>
		<div class="content">
			<transition mode="out-in">
				<!--If keep alive-->
				<keep-alive v-if="keepAliveData">
					<component
						:steps="steps"
						:is="steps[currentStep.index].component"
						:clickedNext="nextButton[currentStep.name]"
						@can-continue="proceed"
						@change-next="changeNextBtnValue"
						:current-step="currentStep"
					></component>
				</keep-alive>
				<!--If not show component and destroy it in each step change-->
				<component
					v-else
					:is="steps[currentStep.index].component"
					:clickedNext="nextButton[currentStep.name]"
					@can-continue="proceed"
					@change-next="changeNextBtnValue"
					:current-step="currentStep"
				></component>
			</transition>
		</div>
		<div :class="['bottom']">
			<div
				v-if="currentStep.index >= 0"
				class="stepper-button previous"
				:class="currentStep.index === 0 ? 'disabled' : ''"
				@click="backStep()"
			>
				<i class="material-icons">keyboard_arrow_left</i>
				<span>{{ 'back' | translate(locale) }}</span>
			</div>
			<div id="totalSavings" v-show="!finalStep">
				<div class="saving-container">
					<p class="savings">${{ savingsPerYer() | numberCommas }}</p>
					<p>Savings Per Year</p>
				</div>

				<div class="saving-container">
					<p class="savings">{{ Math.round(potentialAutomation() * 100) }}%</p>
					<p>Potential Automation Boost</p>
				</div>
			</div>
			<div
				class="stepper-button previous"
				v-show="!finalStep"
				@click="nextStep()"
			>
				<span :class="!finalStep ? '' : 'final'">Next</span>
				<i v-show="!finalStep" class="material-icons">keyboard_arrow_right</i>
			</div>
			<div
				:class="!finalStep ? '' : 'final'"
				class="stepper-button previous"
				v-show="finalStep"
				@click="nextStep()"
			>	
				<a class="talk-to-expert">
					<span style="color=#fff">Talk to an Expert</span>
					<i v-show="!finalStep" class="material-icons">keyboard_arrow_right</i>
				</a>
			</div>
		</div>
	</div>
</template>

<script>
import translations from './Translations.js';
import RoundSlider from 'vue-round-slider';
import roiHelpers from '@/mixins/roiHelpers';

export default {
	mixins: [roiHelpers],

	filters: {
		translate: function(value, locale) {
			return translations[locale][value];
		},
	},
	components: {
		RoundSlider,
	},

	props: {
		locale: {
			type: String,
			default: 'en',
		},
		topButtons: {
			type: Boolean,
			default: false,
		},
		steps: {
			type: Array,
			default: function() {
				return [
					{
						icon: 'mail',
						name: 'first',
						title: 'Sample title 1',
						subtitle: 'Subtitle sample',
					},
					{
						icon: 'report_problem',
						name: 'second',
						title: 'Sample title 2',
						subtitle: 'Subtitle sample',
					},
				];
			},
		},
		keepAlive: {
			type: Boolean,
			default: true,
		},
		reset: {
			type: Boolean,
			default: false,
		},
	},

	data() {
		return {
			currentStep: {},
			previousStep: {},
			nextButton: {},
			canContinue: true,
			finalStep: false,
			keepAliveData: this.keepAlive,
			sliderValue: 1,
		};
	},

	computed: {
		// enterAnimation() {
		//   if (this.currentStep.index < this.previousStep.index) {
		//     return "animated quick fadeInLeft";
		//   } else {
		//     return "animated quick fadeInRight";
		//   }
		// },
		// leaveAnimation() {
		//   if (this.currentStep.index > this.previousStep.index) {
		//     return "animated quick fadeOutLeft";
		//   } else {
		//     return "animated quick fadeOutRight";
		//   }
		// },
		windowWidth() {
			return window.innerWidth;
		},

		savings() {
			this.savingsPerYer();
		},
	},

	methods: {
		activate() {
			if (this.$store.getters.activeStepIndex == '1') {
				return this.$store.getters.activeStepIndex + 1;
			} else if (this.$store.getters.activeStepIndex == '2') {
				return this.$store.getters.activeStepIndex;
			} else {
				return this.$store.getters.activeStepIndex + 1;
			}
		},

		isStepActive(index, step) {
			if (this.currentStep.index === index) {
				return 'activated';
			} else {
				return 'deactivated';
			}
		},

		activateStep(index, back = false) {
			if (this.steps[index]) {
				this.previousStep = this.currentStep;
				this.currentStep = {
					name: this.steps[index].name,
					index: index,
				};

				if (index + 1 === this.steps.length) {
					this.finalStep = true;
				} else {
					this.finalStep = false;
				}

				if (!back) {
					this.$emit('completed-step', this.previousStep);
				}
			}
			this.$emit('active-step', this.currentStep);
		},

		nextStepAction() {
			this.nextButton[this.currentStep.name] = true;
			if (this.canContinue) {
	
				let currentIndex = this.currentStep.index + 1;

				this.activateStep(currentIndex);
			}
			this.canContinue = true;
			this.$forceUpdate();
		},

		nextStep() {
			if (!this.$listeners || !this.$listeners['before-next-step']) {
				this.nextStepAction();
			}

			this.canContinue = true;

			this.$emit('before-next-step', {currentStep: this.currentStep}, (next = true) => {
				this.canContinue = true;
				if (next) {
					this.nextStepAction();
				}
			});
		},

		backStep() {
			this.$emit('clicking-back');
			let currentIndex = this.currentStep.index - 1;
			if (currentIndex >= 0) {
				this.activateStep(currentIndex, true);
			}
			this.canContinue = true;
		},

		proceed(payload) {
			this.canContinue = true;
		},

		changeNextBtnValue(payload) {
			this.nextButton[this.currentStep.name] = payload.nextBtnValue;
			this.$forceUpdate();
		},

		init() {
			// Initiate stepper
			this.activateStep(0);
			this.steps.forEach((step) => {
				this.nextButton[step.name] = false;
			});
		},
	},

	watch: {
		reset(val) {
			if (!val) {
				return;
			}

			this.keepAliveData = false;

			this.init();
			this.previousStep = {};

			this.$nextTick(() => {
				this.keepAliveData = this.keepAlive;
				this.$emit('reset', true);
			});
		},
	},

	created() {
		this.init();
	},
};
</script>

<style src="./HorizontalStepper.scss" scoped lang="scss"></style>
<style scoped>
/* fallback */
@font-face {
	font-family: 'Material Icons';
	font-style: normal;
	font-weight: 400;
	src: local('Material Icons'), local('MaterialIcons-Regular'),
		url(https://fonts.gstatic.com/s/materialicons/v17/2fcrYFNaTjcS6g4U3t-Y5ZjZjT5FdEJ140U2DJYC3mY.woff2)
			format('woff2');
}

.material-icons {
	font-family: 'Material Icons';
	font-weight: normal;
	font-style: normal;
	font-size: 24px;
	line-height: 1;
	letter-spacing: normal;
	text-transform: none;
	display: inline-block;
	white-space: nowrap;
	word-wrap: normal;
	direction: ltr;
	-webkit-font-feature-settings: 'liga';
	-webkit-font-smoothing: antialiased;
}

.final {
	
}
</style>
