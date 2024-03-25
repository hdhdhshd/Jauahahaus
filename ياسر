/**
 * @name GameActivityToggle
 * @author DevilBro
 * @authorId 278543574059057154
 * @version 1.2.1
 * @description Adds a Quick-Toggle Game Activity Button
 * @invite Jx3TjNS
 * @donate https://www.paypal.me/MircoWittrien
 * @patreon https://www.patreon.com/MircoWittrien
 * @website https://mwittrien.github.io/
 * @source https://github.com/mwittrien/BetterDiscordAddons/tree/master/Plugins/GameActivityToggle/
 * @updateUrl https://mwittrien.github.io/BetterDiscordAddons/Plugins/GameActivityToggle/GameActivityToggle.plugin.js
 */

module.exports = (_ => {
	const changeLog = {
		
	};
	
	return !window.BDFDB_Global || (!window.BDFDB_Global.loaded && !window.BDFDB_Global.started) ? class {
		constructor (meta) {for (let key in meta) this[key] = meta[key];}
		getName () {return this.name;}
		getAuthor () {return this.author;}
		getVersion () {return this.version;}
		getDescription () {return `The Library Plugin needed for ${this.name} is missing. Open the Plugin Settings to download it. \n\n${this.description}`;}
		
		downloadLibrary () {
			BdApi.Net.fetch("https://mwittrien.github.io/BetterDiscordAddons/Library/0BDFDB.plugin.js").then(r => {
				if (!r || r.status != 200) throw new Error();
				else return r.text();
			}).then(b => {
				if (!b) throw new Error();
				else return require("fs").writeFile(require("path").join(BdApi.Plugins.folder, "0BDFDB.plugin.js"), b, _ => BdApi.showToast("Finished downloading BDFDB Library", {type: "success"}));
			}).catch(error => {
				BdApi.alert("Error", "Could not download BDFDB Library Plugin. Try again later or download it manually from GitHub: https://mwittrien.github.io/downloader/?library");
			});
		}
		
		load () {
			if (!window.BDFDB_Global || !Array.isArray(window.BDFDB_Global.pluginQueue)) window.BDFDB_Global = Object.assign({}, window.BDFDB_Global, {pluginQueue: []});
			if (!window.BDFDB_Global.downloadModal) {
				window.BDFDB_Global.downloadModal = true;
				BdApi.showConfirmationModal("Library Missing", `The Library Plugin needed for ${this.name} is missing. Please click "Download Now" to install it.`, {
					confirmText: "Download Now",
					cancelText: "Cancel",
					onCancel: _ => {delete window.BDFDB_Global.downloadModal;},
					onConfirm: _ => {
						delete window.BDFDB_Global.downloadModal;
						this.downloadLibrary();
					}
				});
			}
			if (!window.BDFDB_Global.pluginQueue.includes(this.name)) window.BDFDB_Global.pluginQueue.push(this.name);
		}
		start () {this.load();}
		stop () {}
		getSettingsPanel () {
			let template = document.createElement("template");
			template.innerHTML = `<div style="color: var(--header-primary); font-size: 16px; font-weight: 300; white-space: pre; line-height: 22px;">The Library Plugin needed for ${this.name} is missing.\nPlease click <a style="font-weight: 500;">Download Now</a> to install it.</div>`;
			template.content.firstElementChild.querySelector("a").addEventListener("click", this.downloadLibrary);
			return template.content.firstElementChild;
		}
	} : (([Plugin, BDFDB]) => {
		var _this;
		var toggleButton;
		
		const ActivityToggleComponent = class ActivityToggle extends BdApi.React.Component {
			componentDidMount() {
				toggleButton = this;
			}
			render() {
				const enabled = this.props.forceState != undefined ? this.props.forceState : BDFDB.DiscordUtils.getSetting("status", "showCurrentGame");
				delete this.props.forceState;
				return BDFDB.ReactUtils.createElement(BDFDB.LibraryComponents.PanelButton, Object.assign({}, this.props, {
					tooltipText: enabled ? _this.labels.disable_activity : _this.labels.enable_activity,
					icon: iconProps => BDFDB.ReactUtils.createElement(BDFDB.LibraryComponents.SvgIcon, Object.assign({}, iconProps, {
						nativeClass: true,
						width: 20,
						height: 20,
						color: enabled ? "currentColor" : BDFDB.DiscordConstants.ColorsCSS.STATUS_DANGER,
						name: enabled ? BDFDB.LibraryComponents.SvgIcon.Names.GAMEPAD : BDFDB.LibraryComponents.SvgIcon.Names.GAMEPAD_DISABLED
					})),
					onClick: _ => _this.toggle()
				}));
			}
		};
		
		var sounds = [], keybind;
		
		return class GameActivityToggle extends Plugin {
			onLoad () {
				_this = this;
				
				this.defaults = {
					general: {
						showButton:			{value: true,				description: "Show Quick Toggle Button"},
						showItem:			{value: false,				description: "Show Quick Toggle Item"},
						playEnable:			{value: true,				description: "Play Enable Sound"},
						playDisable:			{value: true,				description: "Play Disable Sound"}
					},
					selections: {
						enableSound:			{value: "stream_started",		description: "Enable Sound"},
						disableSound:			{value: "stream_ended",			description: "Disable Sound"}
					}
				};
				
				this.modulePatches = {
					before: [
						"Menu"
					],
					after: [
						"Account"
					]
				};
				
				this.css = `
					${BDFDB.dotCNS._gameactivitytoggleadded + BDFDB.dotCNC.accountinfowithtagasbutton + BDFDB.dotCNS._gameactivitytoggleadded + BDFDB.dotCN.accountinfowithtagless} {
						flex: 1;
						min-width: 0;
					}
				`;
			}
			
			onStart () {
				sounds = [BDFDB.LibraryModules.SoundParser && BDFDB.LibraryModules.SoundParser.keys()].flat(10).filter(n => n).map(s => s.replace("./", "").split(".")[0]).sort();
				
				let cachedState = BDFDB.DataUtils.load(this, "cachedState");
				let state = BDFDB.DiscordUtils.getSetting("status", "showCurrentGame");
				if (!cachedState.date || (new Date() - cachedState.date) > 1000*60*60*24*3) {
					cachedState.value = state;
					cachedState.date = new Date();
					BDFDB.DataUtils.save(cachedState, this, "cachedState");
				}
				else if (cachedState.value != null && cachedState.value != state) BDFDB.DiscordUtils.setSetting("status", "showCurrentGame", cachedState.value);
				
				let SettingsStore = BDFDB.DiscordUtils.getSettingsStore();
				if (SettingsStore) BDFDB.PatchUtils.patch(this, SettingsStore, "updateAsync", {after: e => {
					if (e.methodArguments[0] != "status") return;
					let newSettings = {value: undefined};
					e.methodArguments[1](newSettings);
					if (newSettings.showCurrentGame != undefined) {
						if (toggleButton) toggleButton.props.forceState = newSettings.showCurrentGame.value;
						BDFDB.ReactUtils.forceUpdate(toggleButton);
						BDFDB.DataUtils.save({date: new Date(), value: newSettings.showCurrentGame.value}, this, "cachedState");
					}
				}});
				
				keybind = BDFDB.DataUtils.load(this, "keybind");
				keybind = BDFDB.ArrayUtils.is(keybind) ? keybind : [];
				this.activateKeybind();
				
				BDFDB.DiscordUtils.rerenderAll();
			}
			
			onStop () {
				BDFDB.DiscordUtils.rerenderAll();
			}

			getSettingsPanel (collapseStates = {}) {
				let settingsPanel;
				return settingsPanel = BDFDB.PluginUtils.createSettingsPanel(this, {
					collapseStates: collapseStates,
					children: _ => {
						let settingsItems = [];
						
						for (let key in this.defaults.general) settingsItems.push(BDFDB.ReactUtils.createElement(BDFDB.LibraryComponents.SettingsSaveItem, {
							type: "Switch",
							plugin: this,
							keys: ["general", key],
							label: this.defaults.general[key].description,
							value: this.settings.general[key]
						}));
						
						for (let key in this.defaults.selections) settingsItems.push(BDFDB.ReactUtils.createElement(BDFDB.LibraryComponents.SettingsSaveItem, {
							type: "Select",
							plugin: this,
							keys: ["selections", key],
							label: this.defaults.selections[key].description,
							basis: "50%",
							options: sounds.map(o => ({value: o, label: o.split(/[-_]/g).map(BDFDB.StringUtils.upperCaseFirstChar).join(" ")})),
							value: this.settings.selections[key],
							onChange: value => BDFDB.LibraryModules.SoundUtils.playSound(value, .4)
						}));
						
						settingsItems.push(BDFDB.ReactUtils.createElement("div", {
							className: BDFDB.disCN.settingsrowcontainer,
							children: BDFDB.ReactUtils.createElement("div", {
								className: BDFDB.disCN.settingsrowlabel,
								children: [
									BDFDB.ReactUtils.createElement(BDFDB.LibraryComponents.SettingsLabel, {
										label: "Global Hotkey"
									}),
									BDFDB.ReactUtils.createElement(BDFDB.LibraryComponents.Flex.Child, {
										className: BDFDB.disCNS.settingsrowcontrol + BDFDB.disCN.flexchild,
										grow: 0,
										wrap: true,
										children: BDFDB.ReactUtils.createElement(BDFDB.LibraryComponents.KeybindRecorder, {
											value: !keybind ? [] : keybind,
											reset: true,
											onChange: value => {
												keybind = value;
												BDFDB.DataUtils.save(keybind, this, "keybind")
												this.activateKeybind();
											}
										})
									})
								].flat(10).filter(n => n)
							})
						}));
						
						return settingsItems;
					}
				});
			}
			
			processMenu (e) {
				if (!this.settings.general.showItem || e.instance.props.navId != "account") return;
				let [_, oldIndex] = BDFDB.ContextMenuUtils.findItem(e.instance, {id: BDFDB.ContextMenuUtils.createItemId(this.name, "activity-toggle")});
				if (oldIndex > -1) return;
				let [children, index] = BDFDB.ContextMenuUtils.findItem(e.instance, {id: ["custom-status", "set-custom-status", "edit-custom-status"]});
				if (index > -1) {
					let isChecked = BDFDB.DiscordUtils.getSetting("status", "showCurrentGame");
					children.push(BDFDB.ContextMenuUtils.createItem(BDFDB.LibraryComponents.MenuItems.MenuCheckboxItem, {
						label: BDFDB.LanguageUtils.LanguageStrings.ACTIVITY_STATUS,
						id: BDFDB.ContextMenuUtils.createItemId(this.name, "activity-toggle"),
						icon: _ => BDFDB.ReactUtils.createElement(BDFDB.LibraryComponents.MenuItems.MenuIcon, {
							icon: BDFDB.LibraryComponents.SvgIcon.Names.GAMEPAD
						}),
						showIconFirst: true,
						checked: isChecked,
						action: _ => this.toggle()
					}));
				}
			}
			
			processAccount (e) {
				if (!this.settings.general.showButton) return;
				let [children, index] = BDFDB.ReactUtils.findParent(e.returnvalue, {name: "PanelButton"});
				if (index > -1) {
					e.returnvalue.props.className = BDFDB.DOMUtils.formatClassName(e.returnvalue.props.className, BDFDB.disCN._gameactivitytoggleadded);
					children.unshift(BDFDB.ReactUtils.createElement(ActivityToggleComponent, {}));
				}
			}
			
			activateKeybind () {
				if (keybind && keybind.length) BDFDB.ListenerUtils.addGlobal(this, "GAMEACTIVITY_TOGGLE", keybind, _ => this.toggle());
				else BDFDB.ListenerUtils.removeGlobal(this, "GAMEACTIVITY_TOGGLE");
			}
			
			toggle () {
				const shouldEnable = !BDFDB.DiscordUtils.getSetting("status", "showCurrentGame");
				this.settings.general[shouldEnable ? "playEnable" : "playDisable"] && BDFDB.LibraryModules.SoundUtils.playSound(this.settings.selections[shouldEnable ? "enableSound" : "disableSound"], .4);
				BDFDB.DiscordUtils.setSetting("status", "showCurrentGame", shouldEnable);
			}

			setLabelsByLanguage () {
				switch (BDFDB.LanguageUtils.getLanguage().id) {
					case "bg":		// Bulgarian
						return {
							disable_activity:					"ÐÐµÐ°ÐºÑÐ¸Ð²Ð¸ÑÐ°Ð¹ÑÐµ Ð°ÐºÑÐ¸Ð²Ð½Ð¾ÑÑÑÐ° Ð² Ð¸Ð³ÑÐ°ÑÐ°",
							enable_activity:					"ÐÐºÑÐ¸Ð²Ð¸ÑÐ°Ð¹ÑÐµ Game Activity"
						};
					case "da":		// Danish
						return {
							disable_activity:					"Deaktiver spilaktivitet",
							enable_activity:					"AktivÃ©r spilaktivitet"
						};
					case "de":		// German
						return {
							disable_activity:					"SpieleaktivitÃ¤t deaktivieren",
							enable_activity:					"SpieleaktivitÃ¤t aktivieren"
						};
					case "el":		// Greek
						return {
							disable_activity:					"ÎÏÎµÎ½ÎµÏÎ³Î¿ÏÎ¿Î¯Î·ÏÎ· Î´ÏÎ±ÏÏÎ·ÏÎ¹ÏÏÎ·ÏÎ±Ï ÏÎ±Î¹ÏÎ½Î¹Î´Î¹Î¿Ï",
							enable_activity:					"ÎÎ½ÎµÏÎ³Î¿ÏÎ¿Î¯Î·ÏÎ· Î´ÏÎ±ÏÏÎ·ÏÎ¹ÏÏÎ·ÏÎ±Ï ÏÎ±Î¹ÏÎ½Î¹Î´Î¹Î¿Ï"
						};
					case "es":		// Spanish
						return {
							disable_activity:					"Deshabilitar la actividad del juego",
							enable_activity:					"Habilitar la actividad del juego"
						};
					case "fi":		// Finnish
						return {
							disable_activity:					"Poista pelitoiminto kÃ¤ytÃ¶stÃ¤",
							enable_activity:					"Ota pelitoiminta kÃ¤yttÃ¶Ã¶n"
						};
					case "fr":		// French
						return {
							disable_activity:					"DÃ©sactiver l'activitÃ© de jeu",
							enable_activity:					"Activer l'activitÃ© de jeu"
						};
					case "hr":		// Croatian
						return {
							disable_activity:					"OnemoguÄi aktivnost igre",
							enable_activity:					"OmoguÄi aktivnost u igrama"
						};
					case "hu":		// Hungarian
						return {
							disable_activity:					"Tiltsa le a jÃ¡tÃ©ktevÃ©kenysÃ©get",
							enable_activity:					"EngedÃ©lyezze a jÃ¡tÃ©ktevÃ©kenysÃ©get"
						};
					case "it":		// Italian
						return {
							disable_activity:					"Disabilita l'attivitÃ  di gioco",
							enable_activity:					"Abilita attivitÃ  di gioco"
						};
					case "ja":		// Japanese
						return {
							disable_activity:					"ã²ã¼ã ã¢ã¯ãã£ããã£ãç¡å¹ã«ãã",
							enable_activity:					"ã²ã¼ã ã¢ã¯ãã£ããã£ãæå¹ã«ãã"
						};
					case "ko":		// Korean
						return {
							disable_activity:					"ê²ì íë ë¹íì±í",
							enable_activity:					"ê²ì íë íì±í"
						};
					case "lt":		// Lithuanian
						return {
							disable_activity:					"IÅ¡jungti Å¾aidimÅ³ veiklÄ",
							enable_activity:					"Ä®galinti Å¾aidimÅ³ veiklÄ"
						};
					case "nl":		// Dutch
						return {
							disable_activity:					"Schakel spelactiviteit uit",
							enable_activity:					"Schakel spelactiviteit in"
						};
					case "no":		// Norwegian
						return {
							disable_activity:					"Deaktiver spillaktivitet",
							enable_activity:					"Aktiver spillaktivitet"
						};
					case "pl":		// Polish
						return {
							disable_activity:					"WyÅÄcz aktywnoÅÄ w grach",
							enable_activity:					"WÅÄcz aktywnoÅÄ w grach"
						};
					case "pt-BR":	// Portuguese (Brazil)
						return {
							disable_activity:					"Desativar atividade de jogo",
							enable_activity:					"Habilitar atividade de jogo"
						};
					case "ro":		// Romanian
						return {
							disable_activity:					"DezactivaÈi Activitatea jocului",
							enable_activity:					"ActivaÈi Activitatea jocului"
						};
					case "ru":		// Russian
						return {
							disable_activity:					"ÐÑÐºÐ»ÑÑÐ¸ÑÑ Ð¸Ð³ÑÐ¾Ð²ÑÑ Ð°ÐºÑÐ¸Ð²Ð½Ð¾ÑÑÑ",
							enable_activity:					"ÐÐºÐ»ÑÑÐ¸ÑÑ Ð¸Ð³ÑÐ¾Ð²ÑÑ Ð°ÐºÑÐ¸Ð²Ð½Ð¾ÑÑÑ"
						};
					case "sv":		// Swedish
						return {
							disable_activity:					"Inaktivera spelaktivitet",
							enable_activity:					"Aktivera spelaktivitet"
						};
					case "th":		// Thai
						return {
							disable_activity:					"à¸à¸´à¸à¸à¸²à¸£à¹à¸à¹à¸à¸²à¸à¸à¸´à¸à¸à¸£à¸£à¸¡à¸à¸­à¸à¹à¸à¸¡",
							enable_activity:					"à¹à¸à¸´à¸à¹à¸à¹à¸à¸²à¸à¸à¸´à¸à¸à¸£à¸£à¸¡à¹à¸à¸¡"
						};
					case "tr":		// Turkish
						return {
							disable_activity:					"Oyun EtkinliÄini Devre DÄ±ÅÄ± BÄ±rak",
							enable_activity:					"Oyun EtkinliÄini EtkinleÅtir"
						};
					case "uk":		// Ukrainian
						return {
							disable_activity:					"ÐÐ¸Ð¼ÐºÐ½ÑÑÐ¸ ÑÐ³ÑÐ¾Ð²Ñ Ð°ÐºÑÐ¸Ð²Ð½ÑÑÑÑ",
							enable_activity:					"Ð£Ð²ÑÐ¼ÐºÐ½ÑÑÐ¸ ÑÐ³ÑÐ¾Ð²Ñ Ð°ÐºÑÐ¸Ð²Ð½ÑÑÑÑ"
						};
					case "vi":		// Vietnamese
						return {
							disable_activity:					"Táº¯t hoáº¡t Äá»ng trÃ² chÆ¡i",
							enable_activity:					"Báº­t hoáº¡t Äá»ng trÃ² chÆ¡i"
						};
					case "zh-CN":	// Chinese (China)
						return {
							disable_activity:					"ç¦ç¨æ¸¸ææ´»å¨",
							enable_activity:					"å¯ç¨æ¸¸ææ´»å¨"
						};
					case "zh-TW":	// Chinese (Taiwan)
						return {
							disable_activity:					"ç¦ç¨éæ²æ´»å",
							enable_activity:					"åç¨éæ²æ´»å"
						};
					default:		// English
						return {
							disable_activity:					"Disable Game Activity",
							enable_activity:					"Enable Game Activity"
						};
				}
			}
		};
	})(window.BDFDB_Global.PluginUtils.buildPlugin(changeLog));
})();
