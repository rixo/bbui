<script>
  import { Heading, Body, Button } from "../"
  import { FILE_TYPES } from "./fileTypes"

  const BYTES_IN_KB = 1000
  const BYTES_IN_MB = 1000000

  export let icons = {
    image: "fas fa-file-image",
    code: "fas fa-file-code",
    file: "fas fa-file",
    leftArrow: "fas fa-arrow-left",
    rightArrow: "fas fa-arrow-right",
    deleteButton: "fas fa-times",
    fileUpload: "fas fa-upload",
  }

  export let files = []
  export let fileSizeLimit = BYTES_IN_MB * 20
  export let processFiles
  export let handleFileTooLarge

  let selectedImageIdx = 0
  let fileDragged = false
  // Generate a random ID so that multiple dropzones on the page don't conflict
  let id = Math.random().toString(36).substring(7)

  $: selectedImage = files ? files[selectedImageIdx] : null

  function determineFileIcon(extension) {
    const ext = extension.toLowerCase()

    if (FILE_TYPES.IMAGE.includes(ext)) return icons.image
    if (FILE_TYPES.CODE.includes(ext)) return icons.code

    return icons.file
  }

  async function processFileList(fileList) {
    if (Array.from(fileList).some(file => file.size >= fileSizeLimit)) {
      handleFileTooLarge(fileSizeLimit, file);
      return
    }

    const processedFiles = await processFiles(fileList);

    files = [...processedFiles, ...files]
    selectedImageIdx = 0
  }

  async function removeFile() {
    files.splice(selectedImageIdx, 1)
    files = files
    selectedImageIdx = 0
  }

  function navigateLeft() {
    selectedImageIdx -= 1
  }

  function navigateRight() {
    selectedImageIdx += 1
  }

  function handleFile(evt) {
    processFileList(evt.target.files)
  }

  function handleDragOver(evt) {
    evt.preventDefault()
    fileDragged = true
  }

  function handleDragLeave(evt) {
    evt.preventDefault()
    fileDragged = false
  }

  function handleDrop(evt) {
    evt.preventDefault()
    processFileList(evt.dataTransfer.files)
    fileDragged = false
  }
</script>

<div
  class="dropzone"
  on:dragover={handleDragOver}
  on:dragleave={handleDragLeave}
  on:dragenter={handleDragOver}
  on:drop={handleDrop}
  class:fileDragged>
  <ul>
    {#if selectedImage}
      <li>
        <header>
          <div>
            <i class={determineFileIcon(selectedImage.extension)} />
            <span class="filename">{selectedImage.name}</span>
          </div>
          <p>
            {#if selectedImage.size <= BYTES_IN_MB}
              {selectedImage.size / BYTES_IN_KB}KB
            {:else}{selectedImage.size / BYTES_IN_MB}MB{/if}
          </p>
        </header>
        <div class="delete-button" on:click={removeFile}>
          <i class={icons.deleteButton} />
        </div>
        {#if selectedImageIdx !== 0}
          <div class="nav left" on:click={navigateLeft}>
            <i class={icons.leftArrow} />
          </div>
        {/if}
        <img src={selectedImage.url} />
        {#if selectedImageIdx !== files.length - 1}
          <div class="nav right" on:click={navigateRight}>
            <i class={icons.rightArrow} />
          </div>
        {/if}
      </li>
    {/if}
  </ul>
  <i class={icons.fileUpload} />
  <input {id} type="file" multiple on:change={handleFile} {...$$restProps} />
  <label for={id}>Upload</label>
</div>

<style>
  .dropzone {
    padding: var(--spacing-l);
    border: 2px dashed var(--grey-7);
    text-align: center;
    display: flex;
    align-items: center;
    flex-direction: column;
    border-radius: 10px;
    transition: all 0.3s;
  }

  .fileDragged {
    border: 2px dashed var(--grey-7);
    transform: scale(1.03);
    background: var(--blue-light);
  }

  input[type="file"] {
    display: none;
  }

  label {
    font-family: var(--font-sans);
    cursor: pointer;
    font-weight: 600;
    box-sizing: border-box;
    overflow: hidden;
    border-radius: var(--border-radius-s);
    color: var(--background);
    padding: var(--spacing-s) var(--spacing-l);
    transition: all 0.2s ease 0s;
    display: inline-flex;
    text-rendering: optimizeLegibility;
    min-width: auto;
    outline: none;
    font-feature-settings: "case" 1, "rlig" 1, "calt" 0;
    -webkit-box-align: center;
    user-select: none;
    flex-shrink: 0;
    align-items: center;
    justify-content: center;
    margin-top: 10px;
    width: 100%;
    border: solid 1.5px var(--ink);
    background-color: var(--ink);
  }

  div.nav {
    padding: var(--spacing-xs);
    position: absolute;
    background: black;
    color: white;
    display: flex;
    align-items: center;
    bottom: var(--spacing-s);
    border-radius: 5px;
    transition: 0.2s transform;
  }

  .nav:hover {
    cursor: pointer;
    transform: scale(1.1);
  }

  .left {
    left: var(--spacing-s);
  }

  .right {
    right: var(--spacing-s);
  }

  li {
    position: relative;
    height: 300px;
    background: var(--grey-7);
    display: flex;
    justify-content: center;
    border-radius: 10px;
  }

  img {
    border-radius: 10px;
    width: 100%;
    box-shadow: 0 var(--spacing-s) 12px rgba(0, 0, 0, 0.15);
    object-fit: contain;
  }

  i {
    font-size: 3em;
  }

  .file-icon {
    color: white;
    font-size: 2em;
    margin-right: var(--spacing-s);
  }

  ul {
    padding: 0;
    display: grid;
    grid-gap: var(--spacing-s);
    list-style-type: none;
    width: 100%;
  }

  header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    position: absolute;
    background: linear-gradient(
      180deg,
      rgba(12, 12, 12, 1),
      rgba(60, 60, 60, 0)
    );
    width: 100%;
    border-top-left-radius: 10px;
    border-top-right-radius: 10px;
    height: 60px;
  }

  header > div {
    color: white;
    display: flex;
    align-items: center;
    font-size: 15px;
    margin-left: var(--spacing-m);
    width: 60%;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }

  .filename {
    overflow: hidden;
    margin-left: 5px;
    text-overflow: ellipsis;
  }

  header > p {
    color: var(--grey-5);
    margin-right: var(--spacing-m);
  }

  .delete-button {
    position: absolute;
    top: var(--spacing-s);
    right: var(--spacing-s);
    padding: var(--spacing-s);
    border-radius: 10px;
    opacity: 0;
    transition: all 0.3s;
    color: white;
  }

  .delete-button i {
    font-size: 2em;
  }

  .delete-button:hover {
    opacity: 1;
    cursor: pointer;
    background: linear-gradient(
      to top right,
      rgba(60, 60, 60, 0),
      rgba(255, 0, 0, 0.2)
    );
  }
</style>
