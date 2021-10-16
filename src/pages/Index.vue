<template>
  <q-page>
    <q-input
      v-model="qrLink"
      label="Input value to generate"
      :rules="[(val) => !!val || 'Link field cannot be empty']"
    />
    <br />
    <q-btn color="primary" label="Generate QR Code" @click="generateQrCode" />

    <canvas id="qr-code"></canvas>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
import QRious from "qrious";

export default defineComponent({
  name: "PageIndex",
  data() {
    return {
      qrLink: "",
    };
  },
  methods: {
    generateQrCode: function () {
      if (this.qrLink != "" && this.qrLink != "\n") {
        console.log("generating");

        new QRious({
          level: "H",
          padding: 25,
          size: 300,
          element: document.getElementById("qr-code"),
          value: this.qrLink,
        });
        console.log("generated");
      }
    },
  },
});
</script>
