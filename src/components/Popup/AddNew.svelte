<script lang="ts">
    import { invoke } from "@tauri-apps/api/core";
    import CardApp from "../CardApp.svelte";
    import imageSrc from "../../lib/imageSrc";

    let { modalNew = $bindable(), onAppAdded = () => {} } = $props()

    let appName = $state("")
    let appIconURL = $state("")
    let appPath = $state("")
    let appWorkingDirectory = $state("")

    let submitted = $state(false)

    function close() {
        modalNew = false;
        submitted = false;
        setTimeout(() => {
            appName = ""
            appIconURL = ""
            appPath = ""
            appWorkingDirectory = ""
        }, 500)
    }

    async function addApp() {
        submitted = true;
        if (!appName || !appPath || !appWorkingDirectory) return;

        try {
            await invoke("add_app", {
                app: {
                    name: appName,
                    icon_url: appIconURL,
                    execute_command: appPath,
                    working_directory: appWorkingDirectory,
                }
            });
            modalNew = false;
            submitted = false;
            setTimeout(() => {
                appName = ""
                appIconURL = ""
                appPath = ""
                appWorkingDirectory = ""
            }, 500);
            onAppAdded();
        } catch (e) {
            console.error("Failed to add app:", e);
        }
    }
</script>

<div class="overlay" class:active={modalNew} onclick={() => modalNew = false}></div>
<dialog class="right" class:active={modalNew}>
  <h5>Add a New App</h5>
  <span></span>

  <div class="field label border" class:invalid={submitted && !appName}>
    <input type="text" bind:value={appName}>
    <label>App Name <span style="color: red;">*</span></label>
    <output>The name of the app.</output>
  </div>

  <div class="field label border">
    <input type="text" bind:value={appIconURL}>
    <label>Icon Path/URL</label>
    <output>The path or URL to the icon of the app.</output>
  </div>

  <div class="field label border" class:invalid={submitted && !appPath}>
    <input type="text" bind:value={appPath}>
    <label>Execute Command <span style="color: red;">*</span></label>
    <output>The command to run the app. eg; <code>.\VLC.UTILS.exe</code></output>
  </div>

  <div class="field label border" class:invalid={submitted && !appWorkingDirectory}>
    <input type="text" bind:value={appWorkingDirectory}>
    <label>Working Directory <span style="color: red;">*</span></label>
    <output>The directory to run the app at. eg; <code>D:\Apps\VLC.Utils\</code></output>
  </div>

  <CardApp
      name={appName || "App Name"}
      iconUrl={imageSrc(appIconURL) || "https://placehold.co/128x128"}
      executeCommand=""
      workingDirectory=""
      isPreview={true}
  />

  <nav class="right-align no-space">
    <button class="transparent link" onclick={close}>Close</button>
    <button class="primary link" onclick={addApp}>Add App</button>
  </nav>
</dialog>
