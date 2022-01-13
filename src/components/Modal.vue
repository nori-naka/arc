<template>
  <section id="modalArea" class="modalArea">
    <div id="modalBg" class="modalBg"></div>
    <div class="modalWrapper">
      <div class="modalContents">
        対象のエリアに侵入した際に以下のメッセージを端末に送信します。
        <input type="text" placeholder="Message" ref="msg" :value="polygon_props.value" />
      </div>
      <div class="btContianer">
        <div class="btContianer2">
          <button @click="bt_cancel" class="bt">キャンセル</button>
          <button @click="bt_ok" class="bt">設定</button>
        </div>
      </div>
    </div>
  </section>

</template>

<script>
import { uuidv4 } from './uuidv4';
export default {
  props: [ "polygon_props" ],
  methods: {
    bt_ok() {
      this.$emit("hide_modal", {
        id: this.polygon_props.id,
        msg_id: uuidv4(),
        value: this.$refs.msg.value
      });
      this.$refs.msg.value = "";
    },
    bt_cancel() {
      this.$emit("hide_modal", {
        id: null,
        msg_id: null,
        value: null
      });
      this.$refs.msg.value = "";
    }
  }
}
</script>


<style>
input[type="text"] {
  width: 100%;
  border: 2px solid #aaa;
  border-radius: 4px;
  margin: 8px 0;
  outline: none;
  padding: 8px;
  box-sizing: border-box;
  transition: 0.3s;
}
input[type="text"]:focus {
  border-color: #FFC107;
  box-shadow: 0 0 8px 0 #FFC107;
}
.btContianer {
  display: flex;
}
.btContianer2 {
  margin-left: auto;
}
.bt {
  display: inline-block;
  width: 7rem;
  margin: 5px;
  padding: 7px 20px;
  border-radius: 25px;
  border: 0px;
  text-decoration: none;
  color: #FFF;
  background-image: linear-gradient(45deg, #FFC107 0%, #ff8b5f 100%);
  transition: .4s;
}
.bt:hover {
  background-image: linear-gradient(45deg, #FFC107 0%, #f76a35 100%);
}
.bt:active {
  background-image: linear-gradient(45deg, #9a88ff 0%, #291afa 100%);
}

.modalArea {
  /* display: none; */
  position: fixed;
  z-index: 100; /*サイトによってここの数値は調整 */
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
.modalBg {
  width: 100%;
  height: 100%;
  background-color: rgba(30,30,30,0.9);
}
.modalWrapper {
  position: absolute;
  top: 50%;
  left: 50%;
  transform:translate(-50%,-50%);
  width: 70%;
  max-width: 500px;
  padding: 10px 30px;
  background-color: #fff;
}
</style>