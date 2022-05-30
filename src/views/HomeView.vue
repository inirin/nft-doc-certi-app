<template>
  <!-- Main content -->
  <div class="content">
    <div class="container-fluid">
      <div class="row">
        <div class="col-lg-6">
          <div class="card card-primary card-outline">
            <div class="card-header">
              <h5 class="m-0">입력 문장</h5>
            </div>
            <form @submit.prevent="submit" class="card-body text-right">
              <div class="form-group">
                <textarea
                  v-model="text"
                  class="form-control text-bold"
                  style="font-size: 1.25rem"
                  rows="15"
                  placeholder="Enter ..."
                  required
                ></textarea>
              </div>
              <button type="submit" class="btn btn-primary">Submit</button>
            </form>
          </div>
          <!-- /.card -->
        </div>
        <!-- /.col-md-6 -->
        <div class="col-lg-6">
          <div class="card card-primary card-outline">
            <div class="card-header">
              <h5 class="m-0">검사 결과</h5>
            </div>
            <div class="card-body">
              <div
                v-for="{ text, tokenId } in result"
                :key="tokenId"
                class="form-group"
              >
                <p
                  class="lead text-bold w-100"
                  :class="{ 'text-danger mb-0': tokenId }"
                >
                  {{ text }}
                </p>
                <span class="badge bg-warning">{{ tokenId }}</span>
              </div>
            </div>
          </div>
        </div>
        <!-- /.col-md-6 -->
      </div>
      <!-- /.row -->
    </div>
    <!-- /.container-fluid -->
  </div>
  <!-- /.content -->
</template>

<script lang="ts">
import { defineComponent } from "vue";

import * as IPFS from "ipfs-core";
// eslint-disable-next-line @typescript-eslint/no-var-requires
const CaverExtKAS = require("caver-js-ext-kas");

let ipfs: import("ipfs-core-types").IPFS;
let caver: any;

export default defineComponent({
  data() {
    return {
      text: "Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
      result: [] as {
        text: string;
        tokenId: string;
      }[],
    };
  },
  async mounted() {
    ipfs = await IPFS.create();
    caver = new CaverExtKAS(
      1001,
      "KASPD41DPKDTOTO4REUSBIER",
      "Vo3798Bp7JmBH9lX-Mqocttvs_GC3gsrKBGvvRrN"
    );
  },
  async unmounted() {
    await ipfs.stop();
  },
  methods: {
    async submit() {
      const { cid } = await ipfs.add(this.text, {
        chunker: "rabin-16-32-64",
      });

      // console.log(`[${cid.toString()}] root`);
      this.result = [];

      const links = await ipfs.object.links(cid);
      const hashes = links.map((link) => link.Hash);
      for (const hash of hashes) {
        for await (const chunk of ipfs.cat(hash)) {
          const tokenId = `0x${caver.ipfs.toHex(hash).substring(6, 70)}`;
          try {
            await caver.kas.kip17.mint(
              "docucert1", // addressOrAlias
              "0xE080139Cf553109c7408bc94817505f8aD64D625", // to
              tokenId, // tokenId
              hash.toString() // tokenURI
            );
            const result = { text: chunk.toString(), tokenId: "" };
            this.result.push(result);
          } catch (e) {
            const result = { text: chunk.toString(), tokenId: tokenId };
            this.result.push(result);
          }
        }
      }

      // const ret = await caver.kas.kip17.getToken("docucert1", "0x3");
      // console.log(ret);

      // const result = await caver.kas.kip17.deploy(
      //   "DocuCert1",
      //   "DOCCT",
      //   "docucert1"
      // );
      // const result = await caver.kas.kip17.getTokenList("docucert1");
      // const result = await caver.kas.wallet.createAccount();
      // console.log(result);
      // const contracts = await caver.kas.kip17.getContractList();
      // console.log(contracts);
    },
  },
});
</script>

<style scoped></style>
