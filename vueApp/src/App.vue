<script setup>
import { ref, onMounted } from 'vue'
import { QrcodeStream } from 'vue-qrcode-reader'

// URL
// const URl = 'http://tapxter.test/tags/viewtagownerbyvirtagid'
// const BASE_URL = 'http://tapxter.test'
const PROTOCOL = 'https:'
const BASE_URL = `${PROTOCOL}//tapxter-api-df689ec6366f4275d888.tapxter.com`
const END_POINT = 'tags/viewtagownerbyvirtagid'
const URl = BASE_URL + '/' + END_POINT

// States
const loading = ref(true)
const light = ref(false)
const pauseStatus = ref(false)
const actualVirtualTag = ref('')
const noFrontCamera = ref(false)
const noRearCamera = ref(false)
const facingMode = ref('environment')
const readVirtualTagStatus = ref(1)
const userData = ref([])
const dialogref = ref(null)
//

const onLight = () => {
  light.value = !light.value
}
const unpause = () => {
  pauseStatus.value = false
  readVirtualTagStatus.value = 1
  actualVirtualTag.value = ''
}
//
const onCameraOn = () => {
  loading.value = false
}
const onError = (error) => {
  const triedFrontCamera = facingMode.value === 'user'
  const triedRearCamera = facingMode.value === 'environment'

  const cameraMissingError = error.name === 'OverconstrainedError'

  if (triedRearCamera && cameraMissingError) {
    noRearCamera.value = true
  }

  if (triedFrontCamera && cameraMissingError) {
    noFrontCamera.value = true
  }
  console.error(error)
}
const switchCamera = () => {
  switch (facingMode.value) {
    case 'environment':
      facingMode.value = 'user'
      break
    case 'user':
      facingMode.value = 'environment'
      break
  }
}

const fetchVirtualTag = () => {
  const Barer =
    'q1l439iHRlSv3HaokMblMFhEVUbSN8zcQZrNX36cRe4=.eyJpZGVoYXNoX3VzciI6MywidmlydGFnX2hhc2giOiI3MThmNWM4Zjk4ZjgzIiwicGVyZmlsIjoidXNlciIsImVtYWlsIjoieWlsbWFyLmkuYW1AZ21haWwuY29tIiwibG9naW5fdXNyIjoieWlsbWFyLmkuYW0iLCJkYXRhX3VzciI6IntcIm5vbWJyZVwiOlwiWWlsbWFyIEFsemF0ZSBSb2RyaWd1ZXpcIiwgXCJhcGVsbGlkb3NcIjpcIlwiLCBcInRlbGVmb25vXCI6XCIzMTM3MDYwODgyXCJ9In0=.q1l439iHRlSv3HaokMblMPZs9UhKICK1+UT+sn00YB2iTH74IvEbpjfUVLAAoF0am+202msuoF5tPS43AnlmrIcxrzwxE+Do6qMNAWkIM1lVDES6pVzPd4b4eJiESBHa/RLCGg6j2jvwv1zNdh6fuxTmuHUniosn4S6POgrltcceh5+mG5tnliX7hOlutrXI0pUPCUk1/ONdY/Ym+8+NzZzBHthdztDOM+++eBD4rdHKC4ZdCvid7pcmslb/xKXlWOvNFsJQ0TZ1n7WIc0X0vBS83BvzAFTkZwBvxfNOLpDMWL7Gh87rKTDleYp/zINtaiqqYxdXNRLiVlce8mxfQg=='
  const body = {
    virtagid: actualVirtualTag.value
  }
  return new Promise((resolve, reject) => {
    // console.log(URl, JSON.stringify(body))
    fetch(URl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: 'Bearer ' + Barer
      },
      body: JSON.stringify(body)
    })
      .then((response) => {
        if (response.status === 200) {
          resolve(response.json())
        } else {
          reject(response.json())
        }
      })
      .catch((error) => {
        console.log('ERROR', error)
        reject(error)
      })
  })
}

const onDetect = async (detectedCodes) => {
  const mainQr = detectedCodes[0]
  console.log(mainQr)
  if (mainQr?.rawValue && mainQr?.format === 'qr_code') {
    actualVirtualTag.value = mainQr.rawValue
    pauseStatus.value = true
    try {
      const result = await fetchVirtualTag()
      if (result.data && result.data.length > 0) {
        readVirtualTagStatus.value = 2
        userData.value = [
          { key: 'mail', value: result.data[0].email_acceso }
        ]
        const hiu = JSON.parse(result.data[0].hash_info_usuario ?? '{}')
        for (const key in hiu) {
          userData.value.push({ key, value: hiu[key] })
        }
        // console.log(result.data)
      } else {
        readVirtualTagStatus.value = 3
      }
    } catch (error) {
      console.log('E', error)
      readVirtualTagStatus.value = 3
    }
  }
}

const paintCenterText = (detectedCodes, ctx) => {
  for (const detectedCode of detectedCodes) {
    const {
      boundingBox: { x, y, width, height }
    } = detectedCode

    ctx.lineWidth = 3
    ctx.strokeStyle = '#007bff'
    ctx.strokeRect(x, y, width, height)
    const { boundingBox, rawValue } = detectedCode

    const centerX = boundingBox.x + boundingBox.width / 2
    const centerY = boundingBox.y + boundingBox.height / 2

    const fontSize = Math.max(12, (50 * boundingBox.width) / ctx.canvas.width)

    ctx.font = `bold ${fontSize}px sans-serif`
    ctx.textAlign = 'center'

    ctx.lineWidth = 3
    ctx.strokeStyle = '#fefefe'
    ctx.strokeText(detectedCode.rawValue, centerX, centerY)

    ctx.fillStyle = '#8e24aa'
    ctx.fillText(rawValue, centerX, centerY)
  }
}

onMounted(() => {
})
</script>

<template>
  <div>
    <main class="container">
      <nav>
        <ul>
          <li><strong>Virtag QR</strong></li>
        </ul>
        <ul>
          <li>
            <a target="_blank" href="https://github.com/yebt" class="secondary">
              <svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-brand-github-filled" width="24"
                height="24" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" fill="none" stroke-linecap="round"
                stroke-linejoin="round">
                <path stroke="none" d="M0 0h24v24H0z" fill="none" />
                <path
                  d="M5.315 2.1c.791 -.113 1.9 .145 3.333 .966l.272 .161l.16 .1l.397 -.083a13.3 13.3 0 0 1 4.59 -.08l.456 .08l.396 .083l.161 -.1c1.385 -.84 2.487 -1.17 3.322 -1.148l.164 .008l.147 .017l.076 .014l.05 .011l.144 .047a1 1 0 0 1 .53 .514a5.2 5.2 0 0 1 .397 2.91l-.047 .267l-.046 .196l.123 .163c.574 .795 .93 1.728 1.03 2.707l.023 .295l.007 .272c0 3.855 -1.659 5.883 -4.644 6.68l-.245 .061l-.132 .029l.014 .161l.008 .157l.004 .365l-.002 .213l-.003 3.834a1 1 0 0 1 -.883 .993l-.117 .007h-6a1 1 0 0 1 -.993 -.883l-.007 -.117v-.734c-1.818 .26 -3.03 -.424 -4.11 -1.878l-.535 -.766c-.28 -.396 -.455 -.579 -.589 -.644l-.048 -.019a1 1 0 0 1 .564 -1.918c.642 .188 1.074 .568 1.57 1.239l.538 .769c.76 1.079 1.36 1.459 2.609 1.191l.001 -.678l-.018 -.168a5.03 5.03 0 0 1 -.021 -.824l.017 -.185l.019 -.12l-.108 -.024c-2.976 -.71 -4.703 -2.573 -4.875 -6.139l-.01 -.31l-.004 -.292a5.6 5.6 0 0 1 .908 -3.051l.152 -.222l.122 -.163l-.045 -.196a5.2 5.2 0 0 1 .145 -2.642l.1 -.282l.106 -.253a1 1 0 0 1 .529 -.514l.144 -.047l.154 -.03z"
                  stroke-width="0" fill="currentColor" />
              </svg>
            </a>
          </li>
        </ul>
      </nav>
      <!-- <qrcode-stream @detect="onDetect"></qrcode-stream> -->
      <article>
        <header style="padding: 0 20px">
          <nav>
            <ul>
              <li>
                <a href="#" class="" :class="light ? 'primary' : 'secondary'" @click="onLight">
                  <svg v-if="!light" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                    stroke="currentColor" class="heroicon">
                    <path stroke-linecap="round" stroke-linejoin="round"
                      d="M11.412 15.655L9.75 21.75l3.745-4.012M9.257 13.5H3.75l2.659-2.849m2.048-2.194L14.25 2.25 12 10.5h8.25l-4.707 5.043M8.457 8.457L3 3m5.457 5.457l7.086 7.086m0 0L21 21" />
                  </svg>
                  <svg v-else xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="heroicon">
                    <path fill-rule="evenodd"
                      d="M14.615 1.595a.75.75 0 01.359.852L12.982 9.75h7.268a.75.75 0 01.548 1.262l-10.5 11.25a.75.75 0 01-1.272-.71l1.992-7.302H3.75a.75.75 0 01-.548-1.262l10.5-11.25a.75.75 0 01.913-.143z"
                      clip-rule="evenodd" />
                  </svg>
                </a>
              </li>
            </ul>
            <ul></ul>
            <ul>
              <li>
                <a href="#" class="secondary" @click="switchCamera">
                  <svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-camera-code" width="24"
                    height="24" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" fill="none"
                    stroke-linecap="round" stroke-linejoin="round">
                    <path stroke="none" d="M0 0h24v24H0z" fill="none" />
                    <path
                      d="M11 20h-6a2 2 0 0 1 -2 -2v-9a2 2 0 0 1 2 -2h1a2 2 0 0 0 2 -2a1 1 0 0 1 1 -1h6a1 1 0 0 1 1 1a2 2 0 0 0 2 2h1a2 2 0 0 1 2 2v4" />
                    <path d="M14.948 13.559a3 3 0 1 0 -2.58 2.419" />
                    <path d="M20 21l2 -2l-2 -2" />
                    <path d="M17 17l-2 2l2 2" />
                  </svg>
                </a>
              </li>
            </ul>
          </nav>
        </header>

        <p class="error" v-if="noFrontCamera">
          You don't seem to have a front camera on your device
        </p>
        <p class="error" v-if="noRearCamera">
          You don't seem to have a rear camera on your device
        </p>
        <qrcode-stream class="qrcode-stream" @detect="onDetect" :constraints="{ facingMode }" :torch="light" :formats="['qr_code']"
          :track="paintCenterText" :paused="pauseStatus" @camera-on="onCameraOn" @error="onError">
          <span v-if="loading" aria-busy="true">Generating scanner, please wait…</span>
          <div v-if="pauseStatus && readVirtualTagStatus == 1" class="validation-pending">
            <span aria-busy="true">Validating...</span>
          </div>
          <div v-if="pauseStatus && readVirtualTagStatus == 2" class="validation-success">
            <span>OK</span>
          </div>
          <div v-if="pauseStatus && readVirtualTagStatus == 3" class="validation-failure">
            <span>Error in VT</span>
          </div>
        </qrcode-stream>
        <div>
          <button class="flex-center" @click="unpause" :disabled="!pauseStatus">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="heroicon">
              <path fill-rule="evenodd"
                d="M4.755 10.059a7.5 7.5 0 0112.548-3.364l1.903 1.903h-3.183a.75.75 0 100 1.5h4.992a.75.75 0 00.75-.75V4.356a.75.75 0 00-1.5 0v3.18l-1.9-1.9A9 9 0 003.306 9.67a.75.75 0 101.45.388zm15.408 3.352a.75.75 0 00-.919.53 7.5 7.5 0 01-12.548 3.364l-1.902-1.903h3.183a.75.75 0 000-1.5H2.984a.75.75 0 00-.75.75v4.992a.75.75 0 001.5 0v-3.18l1.9 1.9a9 9 0 0015.059-4.035.75.75 0 00-.53-.918z"
                clip-rule="evenodd" />
            </svg>
            <span>Rescan</span>
          </button>
        </div>
        <footer>
          <div>
            <strong>VT:</strong><code>{{ actualVirtualTag }}</code>
          </div>
          <div>
            <button data-tooltip="See use data" :disabled="readVirtualTagStatus != 2" @click="dialogref.show">
              <svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-user-question" width="24"
                height="24" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor" fill="none" stroke-linecap="round"
                stroke-linejoin="round">
                <path stroke="none" d="M0 0h24v24H0z" fill="none" />
                <path d="M8 7a4 4 0 1 0 8 0a4 4 0 0 0 -8 0" />
                <path d="M6 21v-2a4 4 0 0 1 4 -4h3.5" />
                <path d="M19 22v.01" />
                <path d="M19 19a2.003 2.003 0 0 0 .914 -3.782a1.98 1.98 0 0 0 -2.414 .483" />
              </svg>
            </button>

          </div>
        </footer>
      </article>
      <dialog ref="dialogref">
        <article>
          <header>
            <a href="#close" aria-label="Close" class="close" @click="dialogref.close"></a>
            Use data
          </header>
          <table>
            <tbody>
              <template v-for="dataV in userData" :key="dataV.id">
                <tr>
                  <td><strong>{{ dataV.key }}</strong></td>
                  <td>{{ dataV.value }}</td>
                </tr>
              </template>
            </tbody>
          </table>
        </article>
      </dialog>
    </main>
  </div>
</template>

<style scoped>
.heroicon {
  width: 24px;
  height: 24px;
}

.flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

.validation-success,
.validation-failure,
.validation-pending {
  /* position: absolute; */
  width: 100%;
  height: 100%;
  background-color: rgba(255, 255, 255, 0.8);
  padding: 10px;
  text-align: center;
  font-weight: bold;
  font-size: 1.4rem;
  color: black;

  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
}

.validation-success {
  color: green;
}

.validation-failure {
  color: red;
}

</style>
