<template>
  <q-page class="q-pl-md">
    <q-dialog class="link-input-popup" v-model="qrLinkPopup">
      <q-card class="full-width">
        <q-card-section>
          <!-- <QrLinkPopupComponent :popupText="qrPopupText"/> -->
          <q-input
            label="Insert Link"
            v-model="qrPopupText"
            :rules="[
            (val) => !!val || 'Link field cannot be empty',
             (val) => val.length < 50 || 'Please use shorter links or text',
            ]"
            class="q-py-lg"
          />
          <q-btn
            class="full-width"
            color="primary"
            label="Generate Qr Code"
            @click="generateQrCode"
          ></q-btn>
        </q-card-section>
      </q-card>
    </q-dialog>

    <q-dialog class="qr-code-popup" v-model="qrCodePopup">
      <q-card class="full-width">
        <q-card-section>
          <q-card-section
            class="text-center q-py-lg"
            id="qr-code-loader"
            v-model="qrCodeLoader"
          >
            <div class="q-my-md text-h6">Generating QR Code</div>

            <br />
            <q-spinner-tail color="blue-grey" style="font-size: 6em" />
            <br />
          </q-card-section>

          <div class="text-center vertical-middle">
            <canvas id="qrcode" class=""> </canvas>

            <q-btn
              id="download-qr-code-button"
              class="full-width"
              color="primary"
              label="Download as image"
              @click="downloadQrCode"
            ></q-btn>
          </div>
        </q-card-section>
      </q-card>
    </q-dialog>

    <h4>History</h4>
    <div id="history">
      <div
        class="history-item"
        v-for="linkHistory in qrLinksHistory"
        :key="linkHistory.id"
      >
        <q-list>
          <q-item clickable v-ripple>
            <q-item-section avatar>
              <q-icon color="primary" name="link" />
            </q-item-section>

            <q-item-section>{{ linkHistory.Linkname }}</q-item-section>
          </q-item>
          <q-separator />
        </q-list>
      </div>
    </div>

    <q-page-sticky
      id="new-qr"
      position="bottom-right"
      :offset="[18, 18]"
      @click="showQrLinkPopup"
    >
      <q-btn fab icon="add" color="primary" />
    </q-page-sticky>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
import QRious from "qrious";
import { saveAs } from "file-saver";
import { uid } from "quasar";
import html2canvas from "html2canvas";
import { date } from "quasar";
// import { useQuasar } from 'quasar'

// const $q = useQuasar()
export default defineComponent({
  name: "PageIndex",
  components: {},
  data() {
    return {
      qrLinkPopup: false,
      qrCodePopup: false,
      qrCodeLoader: true,
      qrDownload: false,
      qrPopupText: "",
      qrLinksHistory: [],
    };
  },
  methods: {
    showQrLinkPopup: function () {
      this.qrLinkPopup = true;
    },
    generateQrCode: function () {
      if (this.qrPopupText != "" && this.qrPopupText != "\n") {
        this.qrCodePopup = true;
        this.addLinkToHistory(this.qrPopupText);
        try {
          setTimeout(() => {
            new QRious({
              element: document.getElementById("qrcode"),
              value: this.qrPopupText,
            });
            this.qrCodeLoader = false;
            console.log(this.qrCodeLoader);
            document.getElementById("qr-code-loader").style.display = "none";
            document.getElementById("download-qr-code-button").style.display =
              "block";
            this.qrDownload = true;
            this.qrLinkPopup = false;
            console.log(this.qrPopupText.length)
          }, 3000);
        } catch {
          console.log("err");
        }
      }
      if(this.qrPopupText.length > 50){
        console.log('use shorter characters')
      }
       else {
        console.log("empty");
        //  $q.notify({
        //         message: "Please write some content"
        //       });
      }
    },
    downloadQrCode: function () {
      html2canvas(document.querySelector("#qrcode")).then((canvas) => {
        canvas.toBlob(function (blob) {
          window.saveAs(blob, `qrcode${uid()}.jpg`);
        });
      });
    },
    generateRandomNumber: function (min, max) {
      return Math.floor(Math.random() * (max - min) + min);
    },
    addLinkToHistory: function (link) {
      this.qrLinksHistory.unshift({
        Linkname: link,
        id: uid(),
      });
      this.storeHistory();
      this.updateLinkHistory();
    },
    updateLinkHistory: function () {
      if (!localStorage.getItem("qrLinksHistoryStore")) {
        localStorage.setItem("qrLinksHistoryStore", this.qrLinksHistory);
      } else {
        this.qrLinksHistory = JSON.parse(
          localStorage.getItem("qrLinksHistoryStore")
        );
      }
    },
    storeHistory: function () {
      localStorage.setItem(
        "qrLinksHistoryStore",
        JSON.stringify(this.qrLinksHistory)
      );
    },
  },
  created() {
    this.updateLinkHistory();
  },
});
</script>

<style lang="scss">
#qrcode {
  margin: 0 auto;
}
#download-qr-code-button {
  display: none;
}
</style>
