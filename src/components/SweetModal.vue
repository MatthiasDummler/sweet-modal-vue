<template>
	<!--
		SweetModal for Vue.js
		Sweet, easy and powerful modals and dialogs

		Copyright (c) 2017 Adepto.as AS · Oslo, Norway
		Dual licensed under the MIT and GPL licenses.

		See LICENSE-MIT.txt and LICENSE-GPL.txt
	-->
	<div :class="overlay_classes" v-show="is_open" v-on:click="_onOverlayClick">
		<div :class="modal_classes" :style="modal_style">
			<div class="sweet-box-actions">
				<!-- Custom Actions -->
				<slot name="box-action"></slot>

				<!-- Close Button -->
				<div class="sweet-action-close" v-on:click="close" v-if="!hideCloseButton">
					<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
						<path
							d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z"
							fill="#292c34"
						/>
					</svg>
				</div>
			</div>

			<!-- Title: Housing the title and tabs, if no title is present -->
			<div class="sweet-title" v-if="has_title || has_tabs">
				<!-- Tabs but no title -->
				<template v-if="has_tabs && !has_title">
					<ul class="sweet-modal-tabs">
						<li v-for="tab in tabs" :class="_getClassesForTab(tab)">
							<a href="#" v-on:click.prevent="_changeTab(tab)">
								<div class="sweet-modal-valign">
									<span v-if="tab.icon" v-html="tab.icon" class="sweet-modal-tab-icon" />
									<span class="sweet-modal-tab-title">{{ tab.title }}</span>
								</div>
							</a>
						</li>
					</ul>
				</template>

				<!-- Title -->
				<template v-if="has_title">
					<h2 v-if="title" v-html="title"></h2>
					<slot name="title"></slot>
				</template>
			</div>

			<!-- Tabs: If title AND tabs are present -->
			<ul class="sweet-modal-tabs" v-if="has_title && has_tabs">
				<li v-for="tab in tabs" :class="_getClassesForTab(tab)">
					<a href="#" v-on:click.prevent="_changeTab(tab)">
						<div class="sweet-modal-valign">
							<span v-if="tab.icon" v-html="tab.icon" class="sweet-modal-tab-icon" />
							<span class="sweet-modal-tab-title">{{ tab.title }}</span>
						</div>
					</a>
				</li>
			</ul>

			<!-- Content: Wrapper -->
			<div class="sweet-content" ref="content">
				<!-- Icon: Error -->
				<div class="sweet-modal-icon sweet-modal-error" v-if="icon == 'error'" ref="icon_error">
					<span class="sweet-modal-x-mark">
						<span class="sweet-modal-line sweet-modal-left"></span>
						<span class="sweet-modal-line sweet-modal-right"></span>
					</span>
				</div>

				<!-- Icon: Warning -->
				<div class="sweet-modal-icon sweet-modal-warning" v-if="icon == 'warning'" ref="icon_warning">
					<span class="sweet-modal-body"></span>
					<span class="sweet-modal-dot"></span>
				</div>

				<!-- Icon: Info -->
				<div class="sweet-modal-icon sweet-modal-info" v-if="icon == 'info'" ref="icon_info"></div>

				<!-- Icon: Success -->
				<div class="sweet-modal-icon sweet-modal-success" v-if="icon == 'success'" ref="icon_success">
					<span class="sweet-modal-line sweet-modal-tip"></span>
					<span class="sweet-modal-line sweet-modal-long"></span>
					<div class="sweet-modal-placeholder"></div>
					<div class="sweet-modal-fix"></div>
				</div>

				<!-- Actual Content -->
				<div class="sweet-content-content" v-if="$slots.default">
					<slot></slot>
				</div>
			</div>

			<!-- Buttons -->
			<div class="sweet-buttons" v-if="$slots.button">
				<slot name="button"></slot>
			</div>
		</div>
	</div>
</template>

<script>
export default {
	name: 'SweetModal',

	props: {
		title: {
			type: String,
			required: false,
			default: ''
		},

		overlayTheme: {
			type: String,
			required: false,
			default: 'light'
		},

		modalTheme: {
			type: String,
			required: false,
			default: 'light'
		},

		blocking: {
			type: Boolean,
			required: false,
			default: false
		},

		pulseOnBlock: {
			type: Boolean,
			required: false,
			default: true
		},

		icon: {
			type: String,
			required: false,
			default: ''
		},

		hideCloseButton: {
			type: Boolean,
			required: false,
			default: false
		},

		enableMobileFullscreen: {
			type: Boolean,
			required: false,
			default: true
		},

		width: {
			type: [Number, String],
			required: false,
			default: null
		}
	},

	mounted() {
		this.tabs = this.$children.filter(c => c.cmpName && c.cmpName == 'tab')

		if (this.has_tabs) {
			this.currentTab = this._changeTab(this.tabs[0])
		}

		document.addEventListener('keyup', this._onDocumentKeyup)
	},

	beforeDestroy() {
		this._unlockBody()
		document.removeEventListener('keyup', this._onDocumentKeyup)
	},

	data() {
		return {
			visible: false,
			is_open: false,
			is_bouncing: false,
			tabs: [],

			backups: {
				body: {
					height: null,
					overflow: null
				}
			}
		}
	},

	computed: {
		has_title() {
			return this.title || this.$slots.title
		},

		has_tabs() {
			return this.tabs.length > 0
		},

		has_content() {
			return this.$slots.default
		},

		current_tab() {
			return this.tabs.filter(t => t.active === true)[0]
		},

		overlay_classes() {
			return [
				'sweet-modal-overlay',
				'theme-' + this.overlayTheme,
				'sweet-modal-clickable',
				{
					'is-visible': this.visible,
					blocking: this.blocking
				}
			]
		},

		modal_classes() {
			return [
				'sweet-modal',
				'theme-' + this.modalTheme,
				{
					'has-title': this.has_title,
					'has-tabs': this.has_tabs,
					'has-content': this.has_content,
					'has-icon': this.icon,
					'is-mobile-fullscreen': this.enableMobileFullscreen,
					'is-visible': this.visible,
					'is-alert': (this.icon && !this.has_tabs) || (!this.icon && !this.title && !this.$slots.title),
					bounce: this.is_bouncing,
				}
			]
		},

		modal_style() {
			let width = this.width
			let maxWidth = null

			if (width !== null) {
				if (Number(width) == width) {
					width = width + 'px'
				}

				maxWidth = 'none'
			}

			return {
				width,
				maxWidth
			}
		}
	},

	methods: {
		/**
		 * Open the dialog
		 * Emits an event 'open'
		 *
		 * @param tabId string     Optional id or index of initial tab element.
		 */
		open(tabId = null) {
			if (tabId && this.has_tabs) {
				// Find tab with wanted id.
				let openingTabs = this.tabs.filter((tab) => { return tab.id === tabId })
				if (openingTabs.length > 0) {
					// Set current tab to first match.
					this.currentTab = this._changeTab(openingTabs[0])
				} else {
					// Try opening index instead of id as an alternative.
					let openingTab = this.tabs[tabId]
					if (openingTab) {
						this.currentTab = this._changeTab(openingTab)
					}
				}
			}

			this.is_open = true
			this._lockBody()
			this._animateIcon()

			setTimeout(() => this.visible = true, 30)
			this.$emit('open')
		},

		/**
		 * Close the dialog
		 * Emits an event 'close'
		 */
		close() {
			this.visible = false
			this._unlockBody()

			setTimeout(() => this.is_open = false, 300)
			this.$emit('close')
		},

		/**
		 * Bounce the modal.
		 */
		bounce() {
			this.is_bouncing = true

			setTimeout(() => this.is_bouncing = false, 330)
		},

		/**********************
				INTERNAL METHODS
		 **********************/

		_lockBody() {
			this.backups.body.height = document.body.style.height
			this.backups.body.overflow = document.body.style.overflow

			document.body.style.height = '100%'
			document.body.style.overflow = 'hidden'
		},

		_unlockBody() {
			document.body.style.height = this.backups.body.height
			document.body.style.overflow = this.backups.body.overflow
		},

		_onOverlayClick(event) {
			if (!event.target.classList || event.target.classList.contains('sweet-modal-clickable')) {
				if (this.blocking) {
					if (this.pulseOnBlock) this.bounce()
				} else {
					this.close()
				}
			}
		},

		_onDocumentKeyup(event) {
			if (event.keyCode == 27) {
				if (this.blocking) {
					if (this.pulseOnBlock) this.bounce()
				} else {
					this.close()
				}
			}
		},

		_changeTab(tab) {
			this.tabs.map(t => t.active = t == tab)
			this.currentTab = tab
		},

		_getClassesForTab(tab) {
			return [
				'sweet-modal-tab',

				{
					active: tab.active,
					disabled: tab.disabled
				}
			]
		},

		_animateIcon() {
			if (!this.icon) return

			switch (this.icon) {
				case 'success':
					setTimeout(() => {
						this._applyClasses(this.$refs.icon_success, {
							'': ['animate'],
							'.sweet-modal-tip': ['animateSuccessTip'],
							'.sweet-modal-long': ['animateSuccessLong']
						})
					}, 80)

					break;

				case 'warning':
					this._applyClasses(this.$refs.icon_warning, {
						'': ['pulseWarning'],
						'.sweet-modal-body': ['pulseWarningIns'],
						'.sweet-modal-dot': ['pulseWarningIns']
					})

					break;

				case 'error':
					setTimeout(() => {
						this._applyClasses(this.$refs.icon_error, {
							'': ['animateErrorIcon'],
							'.sweet-modal-x-mark': ['animateXMark']
						})
					}, 80)

					break;
			}
		},

		/**
		 * Apply classes from the classMap to $ref or children of $ref, a native
		 * DOMElement.
		 *
		 * ClassMap:
		 * {
		 *     'selector': [ 'class1', 'class2', ... ]
		 * }
		 *
		 * Empty Selector selects $ref.
		 *
		 * @param DOMNode $ref     Element to apply classes to or children of that element
		 * @param Object  classMap Class Map which elements get which classes (see doc)
		 */
		_applyClasses($ref, classMap) {
			for (let cl in classMap) {
				let classes = classMap[cl]
				let $el

				if (cl == '') {
					$el = $ref
				} else {
					$el = $ref.querySelector(cl)
				}

				$el.classList.remove(...classes)
				$el.classList.add(...classes)
			}
		}
	}
}
</script>

<style>
@keyframes animateSuccessTip {
	0% {
		width: 0;
		left: 1px;
		top: 19px;
	}
	54% {
		width: 0;
		left: 1px;
		top: 19px;
	}
	70% {
		width: 50px;
		left: -8px;
		top: 37px;
	}
	84% {
		width: 17px;
		left: 21px;
		top: 48px;
	}
	100% {
		width: 25px;
		left: 14px;
		top: 45px;
	}
}
@keyframes animateSuccessLong {
	0% {
		width: 0;
		right: 46px;
		top: 54px;
	}
	65% {
		width: 0;
		right: 46px;
		top: 54px;
	}
	84% {
		width: 55px;
		right: 0px;
		top: 35px;
	}
	100% {
		width: 47px;
		right: 8px;
		top: 38px;
	}
}
@keyframes rotatePlaceholder {
	0% {
		transform: rotate(-45deg);
	}
	5% {
		transform: rotate(-45deg);
	}
	12% {
		transform: rotate(-405deg);
	}
	100% {
		transform: rotate(-405deg);
	}
}
.animateSuccessTip {
	animation: animateSuccessTip 0.75s;
}
.animateSuccessLong {
	animation: animateSuccessLong 0.75s;
}
.sweet-modal-icon.sweet-modal-success.animate::after {
	animation: rotatePlaceholder 4.25s ease-in;
}
/* Error Icon */
@keyframes animateErrorIcon {
	0% {
		transform: rotateX(100deg);
		opacity: 0;
	}
	100% {
		transform: rotateX(0deg);
		opacity: 1;
	}
}
.animateErrorIcon {
	animation: animateErrorIcon 0.5s;
}
@keyframes animateXMark {
	0% {
		transform: scale(0.4);
		margin-top: 26px;
		opacity: 0;
	}
	50% {
		transform: scale(0.4);
		margin-top: 26px;
		opacity: 0;
	}
	80% {
		transform: scale(1.15);
		margin-top: -6px;
	}
	100% {
		transform: scale(1);
		margin-top: 0;
		opacity: 1;
	}
}
.animateXMark {
	animation: animateXMark 0.5s;
}
@keyframes pulseWarning {
	0% {
		border-color: #f8d486;
	}
	100% {
		border-color: #f8bb86;
	}
}
.pulseWarning {
	animation: pulseWarning 0.75s infinite alternate;
}
@keyframes pulseWarningIns {
	0% {
		background-color: #f8d486;
	}
	100% {
		background-color: #f8bb86;
	}
}
.pulseWarningIns {
	animation: pulseWarningIns 0.75s infinite alternate;
}
@keyframes rotate-loading {
	0% {
		transform: rotate(0deg);
	}
	100% {
		transform: rotate(360deg);
	}
}
.sweet-modal-icon {
	position: relative;
	width: 80px;
	height: 80px;
	border: 4px solid gray;
	border-radius: 50%;
	margin: auto;
	padding: 0;
	box-sizing: content-box;
}
.sweet-modal-icon.sweet-modal-error {
	border-color: #f44336;
}
.sweet-modal-icon.sweet-modal-error .sweet-modal-x-mark {
	position: relative;
	display: block;
}
.sweet-modal-icon.sweet-modal-error .sweet-modal-line {
	display: block;
	position: absolute;
	top: 37px;
	height: 5px;
	width: 47px;
	background-color: #f44336;
	border-radius: 2px;
}
.sweet-modal-icon.sweet-modal-error .sweet-modal-line.sweet-modal-left {
	transform: rotate(45deg);
	left: 17px;
}
.sweet-modal-icon.sweet-modal-error .sweet-modal-line.sweet-modal-right {
	transform: rotate(-45deg);
	right: 16px;
}
.sweet-modal-icon.sweet-modal-warning {
	border-color: #ff9800;
}
.sweet-modal-icon.sweet-modal-warning .sweet-modal-body {
	position: absolute;
	width: 5px;
	height: 47px;
	left: 50%;
	top: 10px;
	margin-left: -2px;
	border-radius: 2px;
	background-color: #ff9800;
}
.sweet-modal-icon.sweet-modal-warning .sweet-modal-dot {
	position: absolute;
	left: 50%;
	bottom: 10px;
	width: 7px;
	height: 7px;
	margin-left: -3px;
	border-radius: 50%;
	background-color: #ff9800;
}
.sweet-modal-icon.sweet-modal-info {
	border-color: #039be5;
}
.sweet-modal-icon.sweet-modal-info::before {
	content: "";
	position: absolute;
	width: 5px;
	height: 29px;
	left: 50%;
	bottom: 17px;
	margin-left: -2px;
	border-radius: 2px;
	background-color: #039be5;
}
.sweet-modal-icon.sweet-modal-info::after {
	content: "";
	position: absolute;
	width: 7px;
	height: 7px;
	top: 19px;
	margin-left: -3px;
	border-radius: 50%;
	background-color: #039be5;
}
.sweet-modal-icon.sweet-modal-success {
	border-color: #4caf50;
}
.sweet-modal-icon.sweet-modal-success::before,
.sweet-modal-icon.sweet-modal-success::after {
	content: "";
	position: absolute;
	border-radius: 40px;
	width: 60px;
	height: 120px;
	background: white;
	transform: rotate(45deg);
}
.sweet-modal-icon.sweet-modal-success::before {
	border-radius: 120px 0 0 120px;
	top: -7px;
	left: -33px;
	transform: rotate(-45deg);
	-webkit-transform-origin: 60px 60px;
	transform-origin: 60px 60px;
}
.sweet-modal-icon.sweet-modal-success::after {
	border-radius: 0 120px 120px 0;
	top: -11px;
	left: 30px;
	transform: rotate(-45deg);
	-webkit-transform-origin: 0px 60px;
	transform-origin: 0px 60px;
}
.sweet-modal-icon.sweet-modal-success .sweet-modal-placeholder {
	box-sizing: content-box;
	position: absolute;
	left: -4px;
	top: -4px;
	z-index: 2;
	width: 80px;
	height: 80px;
	border: 4px solid rgba(76, 175, 80, 0.2);
	border-radius: 50%;
}
.sweet-modal-icon.sweet-modal-success .sweet-modal-fix {
	position: absolute;
	left: 28px;
	top: 8px;
	z-index: 1;
	width: 7px;
	height: 90px;
	background-color: white;
	transform: rotate(-45deg);
}
.sweet-modal-icon.sweet-modal-success .sweet-modal-line {
	display: block;
	position: absolute;
	z-index: 2;
	height: 5px;
	background-color: #4caf50;
	border-radius: 2px;
}
.sweet-modal-icon.sweet-modal-success .sweet-modal-line.sweet-modal-tip {
	width: 25px;
	left: 14px;
	top: 46px;
	transform: rotate(45deg);
}
.sweet-modal-icon.sweet-modal-success .sweet-modal-line.sweet-modal-long {
	width: 47px;
	right: 8px;
	top: 38px;
	transform: rotate(-45deg);
}
.sweet-modal-icon.sweet-modal-custom {
	border-radius: 0;
	border: none;
	background-size: contain;
	background-position: center center;
	background-repeat: no-repeat;
}
.sweet-modal.theme-dark .sweet-modal-icon.sweet-modal-success::before,
.sweet-modal.theme-dark .sweet-modal-icon.sweet-modal-success::after,
.sweet-modal.theme-dark .sweet-modal-icon.sweet-modal-success .sweet-modal-fix {
	background-color: #182028;
}
.sweet-modal-overlay {
	position: fixed;
	top: 0;
	left: 0;
	width: 100vw;
	height: 100%;
	z-index: 9001;
	font-size: 14px;
	-webkit-font-smoothing: antialiased;
	background: rgba(255, 255, 255, 0.9);
	opacity: 0;
	transition: opacity 0.3s;
	transform: translate3D(0, 0, 0);
	-webkit-perspective: 500px;
}
.sweet-modal-overlay.theme-dark {
	background: rgba(24, 32, 40, 0.94);
}
.sweet-modal-overlay.is-visible {
	opacity: 1;
}
.sweet-modal {
	-moz-box-sizing: border-box;
	box-sizing: border-box;
	background: #fff;
	box-shadow: 0px 8px 46px rgba(0, 0, 0, 0.08), 0px 2px 6px rgba(0, 0, 0, 0.03);
	position: absolute;
	top: 50%;
	left: 50%;
	width: 80%;
	max-width: 640px;
	max-height: 100vh;
	overflow-y: auto;
	border-radius: 2px;
	transform: scale(0.9) translate(calc(-50% - 32px), -50%);
	opacity: 0;
	transition-property: transform, opacity;
	transition-duration: 0.3s;
	transition-delay: 0.05s;
	transition-timing-function: cubic-bezier(0.52, 0.02, 0.19, 1.02);
}
.sweet-modal .sweet-box-actions {
	position: absolute;
	top: 12px;
	right: 12px;
}
.sweet-modal .sweet-box-actions .sweet-action-close {
	display: inline-block;
	cursor: pointer;
	color: #222c38;
	text-align: center;
	width: 42px;
	height: 42px;
	line-height: 42px;
	border-radius: 50%;
}
.sweet-modal .sweet-box-actions .sweet-action-close svg {
	width: 24px;
	height: 24px;
	vertical-align: middle;
	margin-top: -2px;
}
.sweet-modal .sweet-box-actions .sweet-action-close svg path,
.sweet-modal .sweet-box-actions .sweet-action-close svg polygon,
.sweet-modal .sweet-box-actions .sweet-action-close svg rect,
.sweet-modal .sweet-box-actions .sweet-action-close svg circle {
	fill: currentColor;
}
.sweet-modal .sweet-box-actions .sweet-action-close svg {
	fill: currentColor;
}
.sweet-modal .sweet-box-actions .sweet-action-close:hover {
	background: #039be5;
	color: #fff;
}
.sweet-modal .sweet-title {
	text-overflow: ellipsis;
	white-space: nowrap;
	overflow: hidden;
	height: 64px;
	line-height: 64px;
	border-bottom: 1px solid #eaeaea;
	padding-left: 32px;
	padding-right: 64px;
}
.sweet-modal .sweet-title > h2 {
	text-overflow: ellipsis;
	white-space: nowrap;
	overflow: hidden;
	margin: 0;
	padding: 0;
	font-weight: 500;
	font-size: 22px;
}
.sweet-modal ul.sweet-modal-tabs {
	margin: 0;
	padding: 0;
	list-style-type: none;
	display: flex;
	align-items: center;
	width: calc(100% + 32px);
	height: 100%;
	margin-left: -32px;
	overflow-x: auto;
}
.sweet-modal ul.sweet-modal-tabs li.sweet-modal-tab {
	display: block;
	height: 100%;
}
.sweet-modal ul.sweet-modal-tabs li.sweet-modal-tab a {
	text-overflow: ellipsis;
	white-space: nowrap;
	overflow: hidden;
	display: flex;
	align-items: center;
	padding-left: 20px;
	padding-right: 20px;
	color: #222c38;
	text-decoration: none;
	text-align: center;
	height: 100%;
}
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-title {
	display: block;
}
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon {
	display: block;
	line-height: 1;
}
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	svg,
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	img {
	width: 16px;
	height: 16px;
}
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	svg
	path,
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	img
	path,
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	svg
	polygon,
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	img
	polygon,
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	svg
	rect,
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	img
	rect,
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	svg
	circle,
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	img
	circle {
	fill: currentColor;
}
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	svg,
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	img {
	fill: currentColor;
}
.sweet-modal
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	+ span.sweet-modal-tab-title {
	line-height: 1;
	margin-top: 8px;
}
.sweet-modal ul.sweet-modal-tabs li.sweet-modal-tab:first-child a {
	padding-left: 32px;
}
.sweet-modal ul.sweet-modal-tabs li.sweet-modal-tab.active a {
	font-weight: 600;
	color: #039be5;
}
.sweet-modal ul.sweet-modal-tabs li.sweet-modal-tab.disabled a {
	-webkit-user-select: none;
	-moz-user-select: none;
	user-select: none;
	cursor: default;
	pointer-events: none;
	color: #999;
}
.sweet-modal.has-tabs:not(.has-title) .sweet-title {
	height: 84px;
	line-height: 84px;
}
.sweet-modal.has-tabs.has-title ul.sweet-modal-tabs {
	width: 100%;
	height: 48px;
	margin: 0;
	border-bottom: 1px solid #eaeaea;
}
.sweet-modal.has-tabs.has-title ul.sweet-modal-tabs li.sweet-modal-tab a {
	margin-top: -4px;
}
.sweet-modal.has-tabs.has-title
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon {
	display: inline-block;
}
.sweet-modal.has-tabs.has-title
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	svg,
.sweet-modal.has-tabs.has-title
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-icon
	img {
	vertical-align: middle;
	margin-top: -2px;
	margin-right: 8px;
}
.sweet-modal.has-tabs.has-title
	ul.sweet-modal-tabs
	li.sweet-modal-tab
	a
	span.sweet-modal-tab-title {
	display: inline-block;
}
.sweet-modal .sweet-content {
	display: flex;
	align-items: center;
	padding-left: 32px;
	padding-right: 32px;
	padding-top: 24px;
	padding-bottom: 24px;
	line-height: 1.5;
}
.sweet-modal .sweet-content .sweet-content-content {
	flex-grow: 1;
}
.sweet-modal .sweet-content .sweet-modal-tab:not(.active) {
	display: none;
}
.sweet-modal .sweet-content .sweet-modal-icon {
	margin-bottom: 36px;
}
.sweet-modal .sweet-buttons {
	text-align: right;
	padding-left: 20px;
	padding-right: 20px;
	padding-top: 12px;
	padding-bottom: 12px;
}
.sweet-modal .sweet-content + .sweet-buttons {
	border-top: 1px solid #eaeaea;
}
.sweet-modal.is-alert .sweet-content {
	display: block;
	text-align: center;
	font-size: 16px;
	padding-top: 64px;
	padding-bottom: 64px;
}
.sweet-modal.has-tabs.has-icon .sweet-content {
	padding-top: 32px;
	padding-bottom: 32px;
}
.sweet-modal.has-tabs.has-icon .sweet-content .sweet-content-content {
	padding-left: 32px;
}
.sweet-modal.has-tabs.has-icon .sweet-content .sweet-modal-icon {
	margin-bottom: 0;
}
.sweet-modal:not(.has-content) .sweet-modal-icon {
	margin-bottom: 0;
}
.sweet-modal.theme-dark {
	background: #182028;
	color: #fff;
}
.sweet-modal.theme-dark .sweet-box-actions .sweet-action-close {
	color: #fff;
}
.sweet-modal.theme-dark .sweet-title {
	border-bottom-color: #090c0f;
	box-shadow: 0px 1px 0px #273442;
}
.sweet-modal.theme-dark ul.sweet-modal-tabs li a {
	color: #fff;
}
.sweet-modal.theme-dark ul.sweet-modal-tabs li.active a {
	color: #039be5;
}
.sweet-modal.theme-dark ul.sweet-modal-tabs li.disabled a {
	color: #3e5368;
}
.sweet-modal.theme-dark.has-tabs.has-title ul.sweet-modal-tabs {
	border-bottom-color: #090c0f;
	box-shadow: 0px 1px 0px #273442;
}
.sweet-modal.theme-dark .sweet-content + .sweet-buttons {
	border-top-color: #273442;
	box-shadow: 0px -1px 0px #090c0f;
}
.sweet-modal .sweet-buttons,
.sweet-modal .sweet-content {
	opacity: 0;
	transition-property: transform, opacity;
	transition-duration: 0.3s;
	transition-delay: 0.09s;
	transition-timing-function: cubic-bezier(0.52, 0.02, 0.19, 1.02);
}
.sweet-modal .sweet-content {
	transform: translateY(-8px);
}
.sweet-modal .sweet-buttons {
	transform: translateY(16px);
}
.sweet-modal.is-visible {
	transform: translate(-50%, -50%);
	opacity: 1;
}
.sweet-modal.is-visible .sweet-buttons,
.sweet-modal.is-visible .sweet-content {
	transform: none;
	opacity: 1;
}
.sweet-modal.bounce {
	animation-name: bounce;
	animation-duration: 0.3s;
	animation-iteration-count: 2;
	animation-direction: alternate;
}
@media screen and (min-width: 601px) {
	@keyframes bounce {
		0% {
			transform: scale(1) translate(-50%, -50%);
		}
		50% {
			transform: scale(1.02) translate(calc(-50% + 8px), -50%);
		}
		100% {
			transform: scale(1) translate(-50%, -50%);
		}
	}
}
@media screen and (max-width: 600px) {
	.sweet-modal.is-mobile-fullscreen {
		width: 100%;
		height: 100%;
		left: 0;
		top: 0;
		transform: scale(0.9);
	}
	.sweet-modal.is-mobile-fullscreen.is-visible {
		transform: none;
	}
	.sweet-modal.is-mobile-fullscreen .sweet-buttons {
		-moz-box-sizing: border-box;
		box-sizing: border-box;
		position: absolute;
		bottom: 0;
		left: 0;
		width: 100%;
	}
}
</style>
