<script>
  // Import TensorFlow.js and COCO-SSD model
  import * as tf from "@tensorflow/tfjs"
  import * as cocoSsd from "@tensorflow-models/coco-ssd"

  let apiLoading = false
  let error
  let objectCounts = {}
  let result = null
  let imgSrc

  async function countObjects(imageData) {
    apiLoading = true

    const img = new Image()
    img.src = imageData
    imgSrc = imageData
    await new Promise((resolve) => (img.onload = resolve))

    const model = await cocoSsd.load()
    const predictions = await model.detect(img)

    const counts = {}
    predictions.forEach((prediction) => {
      const objectName = prediction.class
      counts[objectName] = (counts[objectName] || 0) + 1
    })

    apiLoading = false
    console.log("counts:", counts)
    return {
      counts,
      total: predictions.length,
    }
  }

  async function handleUpload() {
    const file = fileInput.files[0]

    if (file) {
      const reader = new FileReader()
      reader.onload = async (e) => {
        try {
          const imageData = e.target.result
          objectCounts = await countObjects(imageData)
        } catch (err) {
          error = "Error processing image: " + err.message
          result = null
          objectCounts = null
        }
      }
      // reading the uploaded img
      reader.readAsDataURL(file)
    }
  }

  let fileInput
</script>

<h1>Ai Count objects inside an image</h1>
<label for="">
  upload an image
  <input type="file" name="" id="" bind:this={fileInput} on:change={handleUpload} />
</label>

{#if apiLoading}
  tensorflow ai is loading...
{/if}

{#if error}
  <p>error: {error}</p>
{/if}

{#if objectCounts.counts && objectCounts.total}
  <p>Total objects: {objectCounts.total}</p>
  <h3>AI model has found:</h3>
  {#each Object.entries(objectCounts.counts) as [object, count]}
    <p>{count} of {object}</p>
  {/each}
{/if}

{#if imgSrc}
  <p>Uploaded img:</p>
  <img src={imgSrc} alt="" />
{/if}

<style>
  * {
    box-sizing: border-box;
    padding: 0;
    margin: 0;
  }
  p {
    margin-bottom: 1em;
  }
  img {
    width: 100%;
    max-width: 400px;
  }
</style>
