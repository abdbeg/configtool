<style>
input[type='number'] {
    -moz-appearance: textfield;
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
    -webkit-appearance: none;
}

.b-tooltip {
	pointer-events: none;
}
.center {
  display: flex;
  flex-direction: column;
}
.steps-per-mm-text {
    margin-bottom: 0.4rem;
    text-align: center;
}
.mobile-message {
  display: none;
}

@media (max-width: 767px) {
  .col {
    width: 100%;
  }
  .form-row {
    width: 100%;
    display: table-cell;
  }
  .col-sm-1 {
    max-width: 100%;
  }
  .row {
    display: table-row;
  }
  .long-overflow {
    overflow-x: scroll;
    display: grid;
  }
  .table th, .table td {
    padding-right: 0.6rem;
    padding-left: 0.6rem;
    text-align: center;
  }
  #general .ml-1 {
    margin-left: 0 !important;
  }
  #general .ml-2 {
    margin-left: 0 !important;
  }
  #mapping .col-sm-1 {
    padding: 0;
  }
  .mapping-table-26 {
    width: 26%;
  }
  #motor tr > td  input {
    min-width: 6rem !important;
  }
  .tooltip {
    z-index: 10;
  }
  .col-sm-10 {
    max-width: 100%;
  }
  .steps-per-mm {
    box-shadow: 0px 0px 300px 5px rgba(0,0,0,0.37);
    transition: 0.3s;
  }
  .popover-header {
    padding: 0.75rem;
  }
  .popover-body {
    padding: 0.75rem;
  }
  .form-group {
    margin-bottom: 0.5rem;
  }
  .popover-body .form-row {
    display: inline-flex;
  }
  .popover-body .card-body {
    padding: 1rem;
  }
  .bs-popover-bottom > .arrow::after {
    display: none;
  }
  .bs-popover-bottom > .arrow::before {
    display: none;
  }

}


@media (max-width: 1023px) {
  .mobile-message {
    display: block;
  }
  .container {
    display: none;
  }
  .navbar {
    display: none;
  }
}
</style>


<template>
	<div id="app">
    <h1 class="mobile-message">Unfortunately, configuration tools is not available for mobile devices and some small tablets, please use your PC.</h1>
    <h1 class="mobile-message">К сожалению инструмент для настройки недоступен для мобильных телефонов и маленьких планшетов, пожалуйста используйте компьютер.</h1>
		<b-navbar toggleable="md" variant="light" class="mb-4">
			<b-container>
				<b-navbar-nav>
					<b-nav-item to="Start" :active="$route.path === '/Start'">Start</b-nav-item>
					<b-nav-item to="General" :active="$route.path === '/General'">General</b-nav-item>
					<b-nav-item to="Mapping" :active="$route.path === '/Mapping'">I/O Mapping</b-nav-item>
					<b-nav-item to="Motors" :active="$route.path === '/Motors'">Motors</b-nav-item>
					<b-nav-item to="Endstops" :active="$route.path === '/Endstops'">Endstops</b-nav-item>
					<b-nav-item to="Heaters" :active="$route.path === '/Heaters'">Heaters</b-nav-item>
					<b-nav-item to="Fans" :active="$route.path === '/Fans'">Fans</b-nav-item>
					<b-nav-item to="Tools" :active="$route.path === '/Tools'">Tools</b-nav-item>
					<b-nav-item to="Compensation" :active="$route.path === '/Compensation'">Compensation</b-nav-item>
			<!--		<b-nav-item to="Display" :active="$route.path === '/Display'" v-show="$route.path === '/Display' || board.supportsDisplay">Display</b-nav-item> -->
					<b-nav-item to="Network" :active="$route.path === '/Network'" v-show="$route.path === '/Network' || template.standalone">Network</b-nav-item>
					<b-nav-item to="Finish" :active="$route.path === '/Finish'">Finish</b-nav-item>
				</b-navbar-nav>
			</b-container>
		</b-navbar>

		<b-form ref="mainForm">
			<keep-alive>
				<router-view></router-view>
			</keep-alive>
		</b-form>

		<b-container class="mt-3">
			<ul class="pagination float-left">
				<li class="page-item" :class="{ disabled : isFirstPage }">
					<b-link class="page-link" href="#" @click.prevent="goToPreviousPage">
						« Back
					</b-link>
				</li>
			</ul>

			<ul class="pagination float-right">
				<li class="page-item">
					<b-link class="page-link" href="#" @click.prevent="goToNextPage">
						{{ isLastPage ? 'Finish' : 'Next' }} »
					</b-link>
				</li>
			</ul>
		</b-container>

		<b-modal ref="errorModal" :ok-only="true" title="Invalid input" @hidden="highlightErrors">
			<h4>This page contains errors.</h4>
			<h5>Please fix them before you continue.</h5>
		</b-modal>

		<b-modal ref="ioErrorModal" :ok-only="true" title="Invalid I/O Mapping">
			<h4>Your configuration contains errors.</h4>
			<h5>Check the I/O Mapping page before you continue.</h5>
		</b-modal>

		<b-modal ref="finishModal" size="lg" title="Configuration Ready" @show="finishShow">
			<template v-if="dwcLink || iapLink || rrfLink">
				<p>Put the RepRapFirmware files in the /sys directory and extract Duet Web Control bundle to the /www directory of your SD card. If you are using Duet Web Control, upload those files on the Settings page.</p>
				<b-card bg-variant="light" class="mb-3">
					<ul class="pl-4 mb-0">
						<li v-show="iapLink">
							<a :href="iapLink" target="_blank">
								RepRapFirmware IAP utility for firmware updates
							</a>
						</li>
						<li v-show="rrfLink">
							<a :href="rrfLink" target="_blank">
								RepRapFirmware {{ rrfVersion }}
							</a>
						</li>
						<li v-show="dwcLink">
							<a :href="dwcLink" target="_blank">
								Duet Web Control {{ dwcVersion }}
							</a>
						</li>
						</li>
					</ul>
				</b-card>
			</template>

			<p>The following system files will be generated:</p>
			<b-card ref="finishFiles" bg-variant="light" class="mb-3">
				<span v-if="files.length == 0" v-html="message"></span>
				<ul v-else class="pl-4 mb-0">
					<li v-for="filename in files">
						<b-link @click="showFile(filename)" v-text="filename"></b-link>
					</li>
				</ul>
			</b-card>

			<p>If you are using Duet Web Control, you can upload the ZIP file(s) <u>without extracting</u> on the Settings page. Otherwise you can extract the contents of this configuration bundle directly to the root of your SD card.</p>
			<p>See <a href="https://duet3d.com/wiki/SD_card_folder_structure" target="_blank">this page</a> for further information about the purpose of these files.</p>

			<template slot="modal-footer">
				<a :href="configLink" class="btn btn-secondary" download="config.json">
					<font-awesome-icon icon="save"></font-awesome-icon> Download JSON template
				</a>
				<b-button variant="primary" @click="generateZIP" :disabled="generatingZIP">
					<font-awesome-icon :icon="generatingZIP ? 'hourglass' : 'download'"></font-awesome-icon> {{ generatingZIP ? 'Generating ZIP bundle...' : 'Download configuration bundle as ZIP file' }}
				</b-button>
			</template>
		</b-modal>
	</div>
</template>

<script>
'use strict';

import { mapState, mapMutations } from 'vuex'
import { mapFields } from 'vuex-map-fields'
import saveAs from 'file-saver'

import Compiler from './Compiler.js'
import { goToNextPage, goToPreviousPage } from './Router.js'
import Template from './store/Template.js'

export default {
	computed: {
		...mapState(['board', 'template', 'addDWC', 'addRRF']),
		isFirstPage() { return this.$route.path === '/Start'; },
		isLastPage() { return this.$route.path === '/Finish'; }
	},
	data() {
		return {
			configLink: '#',
			files: [],
			generatingZIP: false,
			message: 'Loading...',
			rrfVersion: null,
			rrfLink: null,
			rrfFile: null,
			iapLink: null,
			iapFile: null,
			dwcVersion: null,
			dwcLink: null,
			dwcFile: null
		}
	},
	methods: {
		...mapMutations(['setTemplate']),
		goToPreviousPage() {
			let pageIndex = this.$router.options.routes.findIndex(route => route.path === this.$route.path);
			if (pageIndex > 0) {
				let prevPage = this.$router.options.routes[--pageIndex];
				if (prevPage.path === '/Network' && !this.template.standalone) {
					prevPage = this.$router.options.routes[--pageIndex];
				}
				if (prevPage.path === '/Display' && !this.board.supportsDisplay) {
					prevPage = this.$router.options.routes[--pageIndex];
				}
				this.$router.push(prevPage);
			}
		},
		goToNextPage() {
			let pageIndex = this.$router.options.routes.findIndex(route => route.path === this.$route.path);
			if (pageIndex + 2 < this.$router.options.routes.length) {
				let nextPage = this.$router.options.routes[++pageIndex];
				if (nextPage.path === '/Display' && !this.board.supportsDisplay) {
					nextPage = this.$router.options.routes[++pageIndex];
				}
				if (nextPage.path === '/Network' && !this.template.standalone) {
					nextPage = this.$router.options.routes[++pageIndex];
				}
				this.$router.push(nextPage);
			} else if (this.$refs.mainForm && this.$refs.mainForm.checkValidity() && this.$refs.mainForm.querySelector('.is-invalid') === null) {
				const templateCopy = JSON.parse(JSON.stringify(this.$store.state.template));
				if (Template.validatePins(templateCopy)) {
					this.$refs.finishModal.show();
				} else {
					this.$refs.ioErrorModal.show();
				}
			} else {
				this.$refs.errorModal.show();
			}
		},
		highlightErrors() {
			this.$refs.mainForm.reportValidity();
		},

		async finishShow() {
			this.rrfVersion = this.rrfLink = this.rrfFile = null;
			this.iapLink = this.iapFile = null;
			this.dwcVersion = this.dwcLink = this.dwcFile = null;

			this.message = 'Loading...';
			this.files = [];
			this.configLink = 'data:text/plain;charset=utf-8,' + encodeURIComponent(JSON.stringify(this.template));

			// Load latest stable RRF version from GitHub
			if (this.addRRF && this.template.standalone) {
				try {
					// Get GitHub list of releases and assets. Do NOT get drafts and prereleases
					const releaseInfo = await Compiler.downloadFile('https://api.github.com/repos/gloomyandy/RepRapFirmware/releases', 'json');
					const firmware = this.template.firmware;
					let latestRelease = null;
					releaseInfo.forEach(function(item) {
						if (!item.draft && !item.prerelease && (!latestRelease || item.created_at > latestRelease.created_at)) {
							if ((firmware < 2 && item.name.indexOf('1.') !== -1) ||
								(firmware >= 2 && firmware < 3 && item.name.indexOf('2.') !== -1) ||
								(firmware >= 3 && item.name.indexOf('3.') !== -1)) {
								latestRelease = item;
							}
						}
					});

					// Attempt to download the required files (IAP+RRF)
					if (latestRelease) {
						const iapFile = (this.template.board === 'duet3') ? (this.template.standalone ? 'Duet3iap_sd_MB6HC.bin' : null) : this.board.iapFile;
						for (let i = 0; i < latestRelease.assets.length; i++) {
							const item = latestRelease.assets[i];
							try {
								if (item.name === this.board.firmwareFile) {
									this.rrfVersion = latestRelease.tag_name;
									this.rrfLink = item.browser_download_url;
									this.rrfFile = await Compiler.downloadFile(item.url, 'blob', 'application/octet-stream');
								}
								else if (item.name === iapFile) {
									this.iapLink = item.browser_download_url;
									this.iapFile = await Compiler.downloadFile(item.url, 'blob', 'application/octet-stream');
								}
							} catch (e) {
								// GitHub cloud servers do not provide CORS headers so the download (probably) fails
							}
						}
					} else {
						throw 'Could not find suitable RepRapFirmware version on GitHub';
					}
				} catch (e) {
					console.warn(`Failed to load RRF from GitHub: ${e}`);
				}
			}

			// Load DWC from the server
			if (this.addDWC && this.template.standalone) {
				try {
					// Get GitHub list of releases and assets. Do NOT get drafts and prereleases
					const releaseInfo = await Compiler.downloadFile('https://api.github.com/repos/abdbeg/DuetWebControl/releases', 'json');
					let latestRelease = null;
					releaseInfo.forEach(function(item) {
						if (!item.draft && !item.prerelease && (!latestRelease || item.created_at > latestRelease.created_at)) {
							latestRelease = item;
						}
					});

					// Attempt to download the required files (IAP+RRF)
					if (latestRelease) {
						for (let i = 0; i < latestRelease.assets.length; i++) {
							const item = latestRelease.assets[i];
							try {
								if (item.name.indexOf('SD') !== -1) {
									this.dwcVersion = latestRelease.tag_name;
									this.dwcLink = item.browser_download_url;
									this.dwcFile = await Compiler.downloadFile(item.url, 'blob', 'application/octet-stream');
									break;
								}
							} catch (e) {
								// GitHub cloud servers do not provide CORS headers so the download (probably) fails
							}
						}
					}
				} catch (e) {
					console.warn(`Failed to load DWC from GitHub: ${e}`);
				}
			}

			// Generate config files
			try {
				const output = await Compiler.compileFile('templates/files.ejs', { template: this.template });
				this.files = output.trim().split('\n');
			} catch (e) {
				this.message = 'Failed to load template from server:<br><br>' + e;
			}
		},
		async generateZIP() {
			this.generatingZIP = true;

			try {
				const zip = await Compiler.compileZIP(this.files, this.template, this.board, this.rrfFile, this.iapFile, this.dwcFile);
				if (Compiler.canDownloadFiles) {
					saveAs(zip, 'config.zip');
				} else {
					alert('Error: This browser does not support blobs! Save your configuration template and try another one.');
				}
			} catch (e) {
				alert('Failed to generate ZIP bundle:\n\n' + e);
			}

			this.generatingZIP = false;
		},
		async showFile(filename) {
			try {
				const output = await Compiler.compileTemplate(filename, this.template, this.board);

				let tab = window.open('about:blank', '_blank');
				if (tab == null) {
					alert('Could not open a new tab!\n\nPlease allow pop-ups for this page and try again.');
				} else {
					tab.document.write(output.replace(/\n/g, '<br>').replace(/ /g, '&nbsp;'));
					tab.document.body.style='font-family: monospace;'
					tab.document.title = filename;
					tab.document.close();
				}
			} catch (e) {
				alert(`Failed to generate file ${filename}:\n\n${e}`);
			}
		}
	},
	mounted() {
		const form = this.$refs.mainForm;
		const errorModal = this.$refs.errorModal;
		this.$router.beforeEach((to, from, next) => {
			if (from.path === '/Mapping' || (form.checkValidity() && form.querySelector('.is-invalid') === null)) {
				// Show error dialog for every page but IO Mapping - there is a discrete check for it anyway
				next();
			} else {
				errorModal.show();
				next(false);
			}
		});

		if (window.jsonTemplate) {
			this.setTemplate({ name: 'existing', data: window.jsonTemplate });
			this.$router.push('/General');
		}
	}
}
</script>
