// ==UserScript==
// @name          Krunker SkidFest
// @description   A full featured Mod menu for game Krunker.io!
// @version       2.28
// @author        SkidLamer - From The Gaming Gurus
// @supportURL    https://discord.gg/AJFXXACdrF
// @homepage      https://skidlamer.github.io/
// @match         *://krunker.io/*
// @exclude       *://krunker.io/editor*
// @exclude       *://krunker.io/social*
// @reqire        https://raw.githubusercontent.com/FireMasterK/BypassAdditions/master/script.user.js
// @updateURL     https://skidlamer.github.io/js/Skidfest.user.js
// @run-at        document-start
// @grant         none
// @noframes
// ==/UserScript==

/* eslint-env es6 */
/* eslint-disable no-caller, no-undef, no-loop-func */

(function(skidStr, CRC2d, skid) {

    class Skid {
        constructor() {
            skid = this;
            this.generated = false;
            this.gameJS = null;
            this.token = null;
            this.downKeys = new Set();
            this.settings = null;
            this.vars = {};
            this.playerMaps = [];
            this.skinCache = {};
            this.inputFrame = 0;
            this.renderFrame = 0;
            this.fps = 0;
            this.lists = {
                renderESP: {
                    off: "Off",
                    walls: "Walls",
                    twoD: "2d",
                    full: "Full"
                },
                renderChams: {
                    off: "Off",
                    "#FFFFFF": "White",
                    "#000000": "Black",
                    "#9400D3": "Purple",
                    "#FF1493": "Pink",
                    "#1E90FF": "Blue",
                    "#0000FF": "DarkBlue",
                    "#00FFFF": "Aqua",
                    "#008000": "Green",
                    "#7FFF00": "Lime",
                    "#FF8C00": "Orange",
                    "#FFFF00": "Yellow",
                    "#FF0000": "Red",
                    Rainbow: "Rainbow",
                },
                autoBhop: {
                    off: "Off",
                    autoJump: "Auto Jump",
                    keyJump: "Key Jump",
                    autoSlide: "Auto Slide",
                    keySlide: "Key Slide"
                },
                autoAim: {
                    off: "Off",
                    correction: "Aim Correction",
                    assist: "Legit Aim Assist",
                    easyassist: "Easy Aim Assist",
                    silent: "Silent Aim",
                    trigger: "Trigger Bot",
                    quickScope: "Quick Scope"
                },
                audioStreams: {
                    off: 'Off',
                    _2000s: 'General German/English',
                    _HipHopRNB: 'Hip Hop / RNB',
                    _Oldskool: 'Hip Hop Oldskool',
                    _Country: 'Country',
                    _Pop: 'Pop',
                    _Dance: 'Dance',
                    _Dubstep: 'DubStep',
                    _Lowfi: 'LoFi HipHop',
                    _Jazz: 'Jazz',
                    _Oldies: 'Golden Oldies',
                    _Club: 'Club',
                    _Folk: 'Folk',
                    _ClassicRock: 'Classic Rock',
                    _Metal: 'Heavy Metal',
                    _DeathMetal: 'Death Metal',
                    _Classical: 'Classical',
                    _Alternative: 'Alternative',
                },
            }
            this.consts = {
                twoPI: Math.PI * 2,
                halfPI: Math.PI / 2,
                playerHeight: 11,
                cameraHeight: 1.5,
                headScale: 2,
                armScale: 1.3,
                armInset: 0.1,
                chestWidth: 2.6,
                hitBoxPad: 1,
                crouchDst: 3,
                recoilMlt: 0.3,
                nameOffset: 0.6,
                nameOffsetHat: 0.8,
            };
            this.key = {
                frame: 0,
                delta: 1,
                xdir: 2,
                ydir: 3,
                moveDir: 4,
                shoot: 5,
                scope: 6,
                jump: 7,
                reload: 8,
                crouch: 9,
                weaponScroll: 10,
                weaponSwap: 11,
                moveLock: 12
            };
            this.css = {
                noTextShadows: `*, .button.small, .bigShadowT { text-shadow: none !important; }`,
                hideAdverts: `#aMerger, #endAMerger { display: none !important }`,
                hideSocials: `.headerBarRight > .verticalSeparator, .imageButton { display: none }`,
                cookieButton: `#onetrust-consent-sdk { display: none !important }`,
                newsHolder: `#newsHolder { display: none !important }`,
            };
            this.isProxy = Symbol("isProxy");
            this.spinTimer = 1800;
            try {
                this.onLoad();
            }
            catch(e) {
                console.error(e);
                console.trace(e.stack);
            }
        }
        canStore() {
            return this.isDefined(Storage);
        }

        saveVal(name, val) {
            if (this.canStore()) localStorage.setItem("kro_utilities_"+name, val);
        }

        deleteVal(name) {
            if (this.canStore()) localStorage.removeItem("kro_utilities_"+name);
        }

        getSavedVal(name) {
            if (this.canStore()) return localStorage.getItem("kro_utilities_"+name);
            return null;
        }

        isType(item, type) {
            return typeof item === type;
        }

        isDefined(object) {
            return !this.isType(object, "undefined") && object !== null;
        }

        isNative(fn) {
            return (/^function\s*[a-z0-9_\$]*\s*\([^)]*\)\s*\{\s*\[native code\]\s*\}/i).test('' + fn)
        }

        getStatic(s, d) {
            return this.isDefined(s) ? s : d
        }

        crossDomain(url) {
            return "https://crossorigin.me/" + url;
        }

        async waitFor(test, timeout_ms = 2e4, doWhile = null) {
            let sleep = (ms) => new Promise((resolve) => setTimeout(resolve, ms));
            return new Promise(async (resolve, reject) => {
                if (typeof timeout_ms != "number") reject("Timeout argument not a number in waitFor(selector, timeout_ms)");
                let result, freq = 100;
                while (result === undefined || result === false || result === null || result.length === 0) {
                    if (doWhile && doWhile instanceof Function) doWhile();
                    if (timeout_ms % 1e4 < freq) console.log("waiting for: ", test);
                    if ((timeout_ms -= freq) < 0) {
                        console.log( "Timeout : ", test );
                        resolve(false);
                        return;
                    }
                    await sleep(freq);
                    result = typeof test === "string" ? Function(test)() : test();
                }
                console.log("Passed : ", test);
                resolve(result);
            });
        };

        async request(url, type, opt = {}) {
            return fetch(url, opt).then(response => {
                if (!response.ok) {
                    throw new Error("Network response from " + url + " was not ok")
                }
                return response[type]()
            })
        }

        async fetchScript() {
            const data = await this.request("https://krunker.io/social.html", "text");
            const buffer = await this.request("https://krunker.io/pkg/krunker." + /\w.exports="(\w+)"/.exec(data)[1] + ".vries", "arrayBuffer");
            const array = Array.from(new Uint8Array(buffer));
            const xor = array[0] ^ '!'.charCodeAt(0);
            return array.map((code) => String.fromCharCode(code ^ xor)).join('');
        }

        createSettings() {
            this.displayStyle = (el, val) => {
                this.waitFor(_=>window[el], 5e3).then(node => {
                    if (node) node.style.display = val ? "none" : "inherit";
                    else console.error(el, " was not found in the window object");
                })
            }
            this.settings = {
                //Rendering
                showSkidBtn: {
                    pre: "<div class='setHed'>Rendering</div>",
                    name: "Show Skid Button",
                    val: true,
                    html: () => this.generateSetting("checkbox", "showSkidBtn", this),
                    set: (value, init) => {
                        let button = document.getElementById("mainButton");
                        if (!this.isDefined(button)) this.createButton("5k1D", "https://i.imgur.com/1tWAEJx.gif", this.toggleMenu, value)
                        else button.style.display = value ? "inherit" : "none";
                    }
                },
                hideAdverts: {
                    name: "Hide Advertisments",
                    val: true,
                    html: () => this.generateSetting("checkbox", "hideAdverts", this),
                    set: (value, init) => {
                        if (value) this.head.appendChild(this.css.hideAdverts)
                        else if (!init) this.css.hideAdverts.remove()
                    }
                },
                hideStreams: {
                    name: "Hide Streams",
                    val: false,
                    html: () => this.generateSetting("checkbox", "hideStreams", this),
                    set: (value) => { this.displayStyle("streamContainer", value) }
                },
                hideMerch: {
                    name: "Hide Merch",
                    val: false,
                    html: () => this.generateSetting("checkbox", "hideMerch", this),
                    set: (value) => { this.displayStyle("merchHolder", value) }
                },
                hideNewsConsole: {
                    name: "Hide News Console",
                    val: false,
                    html: () => this.generateSetting("checkbox", "hideNewsConsole", this),
                    set: (value) => { this.displayStyle("newsHolder", value) }
                },
                hideCookieButton: {
                    name: "Hide Security Manage Button",
                    val: false,
                    html: () => this.generateSetting("checkbox", "hideCookieButton", this),
                    set: (value) => { this.displayStyle("onetrust-consent-sdk", value) }
                },
                noTextShadows: {
                    name: "Remove Text Shadows",
                    val: false,
                    html: () => this.generateSetting("checkbox", "noTextShadows", this),
                    set: (value, init) => {
                        if (value) this.head.appendChild(this.css.noTextShadows)
                        else if (!init) this.css.noTextShadows.remove()
                    }
                },
                customCSS: {
                    name: "Custom CSS",
                    val: "",
                    html: () => this.generateSetting("url", "customCSS", "URL to CSS file"),
                    resources: { css: document.createElement("link") },
                    set: (value, init) => {
                        if (value.startsWith("http")&&value.endsWith(".css")) {
                            //let proxy = 'https://cors-anywhere.herokuapp.com/';
                            this.settings.customCSS.resources.css.href = value
                        }
                        if (init) {
                            this.settings.customCSS.resources.css.rel = "stylesheet"
                            try {
                                this.head.appendChild(this.settings.customCSS.resources.css)
                            } catch(e) {
                                alert(e)
                                this.settings.customCSS.resources.css = null
                            }
                        }
                    }
                },
                renderESP: {
                    name: "Player ESP Type",
                    val: "off",
                    html: () =>
                    this.generateSetting("select", "renderESP", this.lists.renderESP),
                },
                renderTracers: {
                    name: "Player Tracers",
                    val: false,
                    html: () => this.generateSetting("checkbox", "renderTracers"),
                },
                rainbowColor: {
                    name: "Rainbow ESP",
                    val: false,
                    html: () => this.generateSetting("checkbox", "rainbowColor"),
                },
                renderChams: {
                    name: "Player Chams",
                    val: "off",
                    html: () =>
                    this.generateSetting(
                        "select",
                        "renderChams",
                        this.lists.renderChams
                    ),
                },
                renderWireFrame: {
                    name: "Player Wireframe",
                    val: false,
                    html: () => this.generateSetting("checkbox", "renderWireFrame"),
                },
                customBillboard: {
                    name: "Custom Billboard Text",
                    val: "",
                    html: () =>
                    this.generateSetting(
                        "text",
                        "customBillboard",
                        "Custom Billboard Text"
                    ),
                },
                //Weapon
                autoReload: {
                    pre: "<br><div class='setHed'>Weapon</div>",
                    name: "Auto Reload",
                    val: false,
                    html: () => this.generateSetting("checkbox", "autoReload"),
                },
                autoAim: {
                    name: "Auto Aim Type",
                    val: "off",
                    html: () =>
                    this.generateSetting("select", "autoAim", this.lists.autoAim),
                },
                frustrumCheck: {
                    name: "Line of Sight Check",
                    val: false,
                    html: () => this.generateSetting("checkbox", "frustrumCheck"),
                },
                wallPenetrate: {
                    name: "Aim through Penetratables",
                    val: false,
                    html: () => this.generateSetting("checkbox", "wallPenetrate"),
                },
                weaponZoom: {
                    name: "Weapon Zoom",
                    val: 1.0,
                    min: 0,
                    max: 50.0,
                    step: 0.01,
                    html: () => this.generateSetting("slider", "weaponZoom"),
                    set: (value) => { if (this.renderer) this.renderer.adsFovMlt = value;}
                },
                weaponTrails: {
                    name: "Weapon Trails",
                    val: false,
                    html: () => this.generateSetting("checkbox", "weaponTrails"),
                    set: (value) => { if (this.me) this.me.weapon.trail = value;}
                },
                //Player
                autoBhop: {
                    pre: "<br><div class='setHed'>Player</div>",
                    name: "Auto Bhop Type",
                    val: "off",
                    html: () => this.generateSetting("select", "autoBhop", this.lists.autoBhop),
                },
                thirdPerson: {
                    name: "Third Person",
                    val: false,
                    html: () => this.generateSetting("checkbox", "thirdPerson"),
                    set: (value, init) => {
                        if (value) this.thirdPerson = 1;
                        else if (!init) this.thirdPerson = undefined;
                    }
                },
                skinUnlock: {
                    name: "Unlock Skins",
                    val: false,
                    html: () => this.generateSetting("checkbox", "skinUnlock", this),
                },
                //GamePlay
                disableWpnSnd: {
                    pre: "<br><div class='setHed'>GamePlay</div>",
                    name: "Disable Players Weapon Sounds",
                    val: false,
                    html: () => this.generateSetting("checkbox", "disableWpnSnd", this),
                },
                disableHckSnd: {
                    name: "Disable Hacker Fart Sounds",
                    val: false,
                    html: () => this.generateSetting("checkbox", "disableHckSnd", this),
                },
                autoActivateNuke: {
                    name: "Auto Activate Nuke",
                    val: false,
                    html: () => this.generateSetting("checkbox", "autoActivateNuke", this),
                },
                autoFindNew: {
                    name: "New Lobby Finder",
                    val: false,
                    html: () => this.generateSetting("checkbox", "autoFindNew", this),
                },
                autoClick: {
                    name: "Auto Start Game",
                    val: false,
                    html: () => this.generateSetting("checkbox", "autoClick", this),
                },
                inActivity: {
                    name: "No InActivity Kick",
                    val: true,
                    html: () => this.generateSetting("checkbox", "autoClick", this),
                },
                //Radio Stream Player
                playStream: {
                    pre: "<br><div class='setHed'>Radio Stream Player</div>",
                    name: "Stream Select",
                    val: "off",
                    html: () => this.generateSetting("select", "playStream", this.lists.audioStreams),
                    set: (value) => {
                        if (value == "off") {
                            if ( this.settings.playStream.audio ) {
                                this.settings.playStream.audio.pause();
                                this.settings.playStream.audio.currentTime = 0;
                                this.settings.playStream.audio = null;
                            }
                            return;
                        }
                        let url = this.settings.playStream.urls[value];
                        if (!this.settings.playStream.audio) {
                            this.settings.playStream.audio = new Audio(url);
                            this.settings.playStream.audio.volume = this.settings.audioVolume.val||0.5
                        } else {
                            this.settings.playStream.audio.src = url;
                        }
                        this.settings.playStream.audio.load();
                        this.settings.playStream.audio.play();
                    },
                    urls: {
                        _2000s: 'http://0n-2000s.radionetz.de/0n-2000s.aac',
                        _HipHopRNB: 'https://stream-mixtape-geo.ntslive.net/mixtape2',
                        _Country: 'https://live.wostreaming.net/direct/wboc-waaifmmp3-ibc2',
                        _Dance: 'http://streaming.radionomy.com/A-RADIO-TOP-40',
                        _Pop: 'http://bigrradio.cdnstream1.com/5106_128',
                        _Jazz: 'http://strm112.1.fm/ajazz_mobile_mp3',
                        _Oldies: 'http://strm112.1.fm/60s_70s_mobile_mp3',
                        _Club: 'http://strm112.1.fm/club_mobile_mp3',
                        _Folk: 'https://freshgrass.streamguys1.com/irish-128mp3',
                        _ClassicRock: 'http://1a-classicrock.radionetz.de/1a-classicrock.mp3',
                        _Metal: 'http://streams.radiobob.de/metalcore/mp3-192',
                        _DeathMetal: 'http://stream.laut.fm/beatdownx',
   
