<!--
Contributors: Arsalan Ali Daim

Description: This component handles the decoding/retrival of information from a QR Code and generating a new QR agaisnt the provided information.
-->
<template>
  <modal
    name="qr-camera-modal"
    height="auto"
    classes="p-2"
    :clickToClose="false"
  >
    <h1 class="view-name">QR Code Scanner</h1>
    <qrcode-drop-zone @detect="onFileDrag">
      <div
        class="drop-area h-full drop-zone-border"
        @dragleave="setDragValue(false)"
        @dragover="setDragValue(true)"
        :class="{ dragover: dragover }"
      >
        <div class="view address-book">
          <div @click.stop>
            <!-- QR Scanner Modal -->
            <div
              class="flex flex-col items-center justify-center p-10"
              :class="[dragover ? dragover : 'bg-white']"
            >
              <div style="width: 50px">
                <qr-code-icon style="fill: #38a169" class="icon" />
              </div>
              <p class="text-xl my-2" v-if="openPath === 'decode'">
                Please scan a QR code or drag/drop a file to import
              </p>
              <p class="text-xl my-2" v-if="openPath === 'create'">
                Please provide the information to create or drag/drop a file to
                import a QR code
              </p>
            </div>
            <div
              class="flex flex-col items-center justify-center"
              v-if="openPath === 'decode'"
            >
              <qrcode-stream
                v-if="qrCameraModalOpen"
                class="qr-code rounded-lg"
                style="border-radius: 0.5rem !important"
                @init="onQrInit"
                @decode="onQrDecode"
              >
                <div v-if="qrLoading" class="loading-indicator text-center">
                  Loading...
                </div>
              </qrcode-stream>
            </div>

            <!-- User Input section for QR generation -->
            <div
              class="flex flex-col items-center justify-center"
              v-if="openPath === 'create'"
            >
              <div class="input-field col-md-6 pl-0">
                <label for="settings_user-fname">QR Code Information</label>
                <input
                  id="qr_info"
                  v-model="qrCodeInfo"
                  class="form-control"
                  type="text"
                />
              </div>
            </div>
          </div>
        </div>
      </div>
    </qrcode-drop-zone>
    <!-- action buttons -->
    <div class="flex items-center justify-center pt-10">
      <button
        v-if="openPath === 'create'"
        v-ripple="'rgba(255, 255, 255, .2)'"
        class="btn text-white bg-red-600"
        required
        @click.stop="submitInfo()"
      >
        Submit
      </button>
      <button
        v-ripple="'rgba(255, 255, 255, .2)'"
        class="btn text-white bg-red-600"
        @click.stop="closeUploadModal"
      >
        Cancel
      </button>
    </div>
  </modal>
</template>

<script>
// Components
import QrCodeIcon from '../icons/QrCode.vue'

import ClickOutside from 'vue-click-outside'

// Utils
import { mapState } from 'vuex'

const initState = () => ({
  qrCodeInfo: '',
  qrLoading: true,
  dragover: false
})

export default {
  name: 'QRCodeReader',
  components: {
    QrCodeIcon
  },
  directives: {
    ClickOutside
  },
  computed: {
    ...mapState({
      user: (state) => state.accounts.active
    })
  },
  props: {
    qrCameraModalOpen: {
      type: Boolean,
      default: () => {
        return false
      }
    },
    openPath: {
      type: String,
      default: () => {
        return 'decode'
      }
    }
  },
  data: initState,
  watch: {
    qrCameraModalOpen() {
      this.dragover = false
      if (this.qrCameraModalOpen) {
        this.$modal.show('qr-camera-modal')
      } else {
        this.$modal.hide('qr-camera-modal')
      }
    }
  },
  mounted() {},
  methods: {
    setDragValue(val) {
      this.dragover = val
    },
    closeUploadModal() {
      this.dragover = false
      this.$emit('close-upload-modal')
    },
    qrParser(data) {
      try {
        this.$toast.success('QR Code imported successfully!')
        this.dragover = false
        this.$emit('qr-imported', data)
      } catch (error) {
        this.$toast.error(
          'An error occured while adding the scanned QR Code.',
          ''
        )
      }
    },
    submitInfo() {
      const { $toast } = this
      if (this.qrCodeInfo.trim() === '') {
        $toast.error(
          'Please enter the required information. Before proceeding!',
          '',
          {
            position: 'center',
            timeout: 1000
          }
        )
      } else {
        this.dragover = false
        this.$emit('qr-imported', this.qrCodeInfo)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.qr-code {
  border-radius: 0.5rem !important;
  width: 400px !important;
  height: 350px !important;
}
.dragover {
  background-color: rgba(56, 161, 105, 0.1);
  z-index: 100 !important;
}
.drop-zone-border {
  border: 2px dashed;
}
</style>
