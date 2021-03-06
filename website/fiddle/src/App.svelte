<script>
  import * as luigiCorePkg from '../node_modules/@luigi-project/core/package.json';
  import defaultConfig from './defaultConfig.js';
  import { onMount } from 'svelte';

  export let luigiVersion = luigiCorePkg.version;

  let defaultConfigString = defaultConfig;
  let configString = defaultConfigString;

  function exec(jsString) {
    return eval(jsString);
  }

  function reloadConfig() {
	let customConfig = sessionStorage.getItem('fiddle');
	let customConfigPreviousSession = localStorage.getItem('fiddle');
	if(!customConfig && customConfigPreviousSession) {
		if (confirm('We found a fiddle from a previous session. Do you want to restore it?')) {
			customConfig = customConfigPreviousSession;
        	sessionStorage.setItem('fiddle', customConfig);
		}
	}

    try {
      if (!customConfig) {
        sessionStorage.setItem('fiddle', defaultConfigString);
        customConfig = defaultConfigString;
      }
      exec(customConfig);
      configString = customConfig;
    } catch (e) {
      console.error(e);
      sessionStorage.removeItem('fiddle');
      exec(defaultConfigString);
      configString = defaultConfigString;
    }
  }

  function saveConfig() {
    localStorage.setItem('fiddle', window.editor.getValue());
    sessionStorage.setItem('fiddle', window.editor.getValue());
    window.location.href = '/';
  }

  function saveConfigTA() {
    localStorage.setItem('fiddle', window.editorTA.value);
    sessionStorage.setItem('fiddle', window.editorTA.value);
    window.location.href = '/';
  }

  function closeConfig() {
    document.body.classList.remove('editorVisible');
  }

  function resetConfig() {
	window.editor.setValue(defaultConfigString);
    window.editorTA.textContent = defaultConfigString;
    window.editor.clearSelection();
  }

   function openConfig() {
	window.editor.setValue(configString);
	window.editorTA.textContent = configString;
    window.editor.clearSelection();
    document.body.classList.add('editorVisible');
  }

  function hide() {
    document.body.classList.remove('show-bar');
  }

  function clearAll() {
    if (confirm('Do you really want to clear all stored data?')) {
      window.location.href = '/secureLeave.html';
    }
  }

  onMount(async () => {
	window.editor = ace.edit('editor');
	window.editorTA = document.getElementById('editorTA');
    editor.session.setMode('ace/mode/javascript');
    reloadConfig();
  });
</script>

<style>
  :global(.show-bar) .fiddle-toolbar {
    display: block;
  }

  :global(.show-bar) #app .fd-app__sidebar,
  :global(.show-bar) .fd-page.iframeContainer {
    bottom: 50px !important;
  }

  :global(body) {
    font-family: var(--sapFontFamily, "72", "72full", Arial, Helvetica, sans-serif);
  }

  :global(#ext-cookiebar) {
    width: 100%;
    position: fixed;
    top: 0;
    left: 0;
    padding: 10px 5px;
    background: rgba(0,0,0,.6);
    font-size: 12px;
    color: white;
    text-align: center;
    box-sizing: border-box;
    z-index: 1000;
  }

  :global(#ext-cookiebar a) {
    color: #2deb8a;
  }

  :global(#ext-cookiebar a:hover) {
    text-decoration: none;
  }

  .fiddle-toolbar {
    border-top: 1px solid rgb(48, 48, 48);
    background: #3c4553;
    position: fixed;
    bottom: 0;
    height: 49px;
    width: 100%;
    z-index: 1;
    display: none;
  }

  .fiddle-toolbar .fd-link {
    color: #2deb8a;
  }

  .editor_container {
    visibility: hidden;
    z-index: -1;
    width: 100%;
    height: 100%;
  }

  :global(.editorVisible) .editor_container {
    visibility: visible;
    z-index: 2;
  }

  #editor, #editorTA {
    width: 100%;
    height: calc(100vh - 300px);
  }

  #editorTA {
	  font-family: monospace;
	  padding: 0;
  }

  .fd-action-bar__header {
    overflow: visible;
    padding-left: 5px;
    color: white;
    font-size: 0.8em;
    margin-top: -3px;
  }

  .fd-action-bar__header a.fd-link {
    margin-left: 10px;
  }

  .fd-action-bar > .fd-action-bar__actions {
    margin: -3px 5px 2px -5px;
  }

  .fd-action-bar__header img {
    vertical-align: middle;
  }

  .btn-primary {
    display: inline-block;
    border: 2px solid #2deb8a;
    border-radius: 8px;
    -webkit-box-shadow: 0 8px 24px -17px #000101;
    box-shadow: 0 8px 24px -17px #000101;
    background-color: #2deb8a;
    font-size: 12px;
    font-weight: 600;
    font-family: 'Open Sans', sans-serif;
    color: #29303a;
    cursor: pointer;
    -webkit-transition: all 0.2s ease;
    transition: all 0.2s ease;
  }

  .btn-primary:hover,
  .btn-primary:active,
  .btn-primary:focus {
    background-color: white;
    color: #2deb8a;
  }

  @media (max-width: 600px) {
    .lui-mobile-hide {
      display: none;
    }
  }
  @media (min-width: 600px) {
    .lui-mobile-show {
      display: none;
    }
  }
</style>

<div class="editor_container fd-shell__overlay fd-overlay fd-overlay--modal">
  <div class="fd-modal" role="dialog" style="width:80%; max-width:80%;">
    <div class="fd-modal__content" role="document">
      <header class="fd-modal__header">
        <h1 class="fd-modal__title">Luigi Config</h1>
        <button
          class="fd-button--light fd-modal__close"
          on:click={closeConfig} />
      </header>
      <div class="fd-modal__body_nostyle">
        <div id="editor" class="lui-mobile-hide"/>
		<textarea id="editorTA" class="lui-mobile-show"/>
      </div>
      <footer class="fd-modal__footer">
        <div class="fd-modal__actions">
          <button class="fd-button--light" on:click={resetConfig}>Reset</button>
          <button class="fd-button lui-mobile-hide" on:click={saveConfig}>Apply</button>
          <button class="fd-button lui-mobile-show" on:click={saveConfigTA}>Apply</button>
          <button class="fd-button--light" on:click={closeConfig}>Cancel</button>
        </div>
      </footer>
    </div>
  </div>
</div>

<div class="fiddle-toolbar">
  <div class="fd-action-bar">
    <div class="fd-action-bar__header">
      <img alt="Luigi" src="./img/luigi.png" />
      <span class="lui-mobile-hide">powered by Luigi v{luigiVersion}</span>
      <span>
        <a
          class="fd-link"
          href="https://www.sap.com/about/legal/privacy.html"
          target="_blank">
          Privacy Policy
        </a>
        <a
          class="fd-link"
          href="https://www.sap.com/about/legal/impressum.html"
          target="_blank">
          Legal
        </a>
      </span>
    </div>
    <div class="fd-action-bar__actions">
      <button class="fd-button--standard btn-primary" on:click={clearAll}>
        <span class="lui-mobile-hide">Clear All</span>
        <span class="lui-mobile-show sap-icon--delete" />
      </button>
      <span class="lui-sep" />
      <button class="fd-button--standard btn-primary" on:click={openConfig}>
        <span class="lui-mobile-hide">Modify Config</span>
        <span class="lui-mobile-show sap-icon--edit" />
      </button>
      <button class="fd-button--standard btn-primary" on:click={hide}>
        <span class="lui-mobile-hide">Hide</span>
        <span class="lui-mobile-show sap-icon--hide" />
      </button>
    </div>
  </div>
</div>
