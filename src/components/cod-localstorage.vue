<template>
  <div>
    <div class="container card1">
      <label for="justificativa" class="label-justificativa">Justificativa</label>
      <input
        id="justificativa"
        type="text"
        v-model="inputText"
        maxlength="150"
        placeholder="Digite um motivo para fazer opt out" />
      <span class="input_length"
        ><span>{{ inputText.length }}</span
        >/150 caracteres</span
      >
      <span class="label-check-box"
        ><input type="checkbox" v-model="hasObservationCheckbox" /> Permitir que o usuário informe uma observação.</span
      >

      <button class="input-submit" @click="submitContainer1">Salvar alterações</button>
    </div>
    <div class="container card2" v-if="localStorageItems.length > 0">
      <div class="coluna_esquerda">
        <span class="title"
          ><p>Nós sentiremos sua falta</p>
          <img alt="Vue logo" src="./img/img.svg"
        /></span>
        <span class="msg_user">
          Lamentamos ver você indo embora. <br />Para que possamos <span>melhorar nossa comunicação</span> , gostaríamos de
          <span>saber os motivos</span> que o levaram a tomar essa decisão.
          <h3 class="opcional">
            <span>O questionário é opcional.</span>
          </h3>
        </span>
      </div>
      <div class="coluna_direita">
        <span>
          <p class="title">Por que você quer se descadastrar?</p>
        </span>
        <ul>
          <li class="itemJustificativa" v-for="(item, index) in localStorageItems" :key="index">
            <div>
              <input type="radio" v-model="selectedItemIndex" :value="index" />
              {{ item.text }}
            </div>
            <input
              id="justificativa"
              type="text"
              v-if="item.hasObservation && index === selectedItemIndex"
              v-model="item.observation"
              maxlength="150"
              @input="updateInputLength(index)" />
            <span class="input_length" v-if="item.hasObservation && index === selectedItemIndex">
              <span>{{ inputLengths[index] }}</span
              >/150 caracteres
            </span>
          </li>
        </ul>

        <span>
          <p class="label-descadastramento-email">O descadastramento será efetuado para o-email:</p>
          <p class="email-descadastrado">nome.sobrenome@pmweb.com.br</p>
        </span>
        <button class="submit-descadastro" @click="submitContainer2">Descadastrar</button>
      </div>
    </div>
  </div>
</template>

<script>
import "./styles/container1.css";
import "./styles/container2.css";
import "./styles/global.css";

export default {
  data() {
    return {
      inputText: "",
      hasObservationCheckbox: false,
      localStorageItems: [],
      selectedItemIndex: -1,
      inputLengths: [],
    };
  },
  methods: {
    submitContainer1() {
      const text = this.inputText;
      const item = {
        text: text,
        hasObservation: this.hasObservationCheckbox,
        observation: "",
      };
      this.localStorageItems.push(item);
      localStorage.setItem("items", JSON.stringify(this.localStorageItems));
      this.inputText = "";
    },
    submitContainer2() {
      console.log(this.localStorageItems);
    },
    loadLocalStorageItems() {
      const items = localStorage.getItem("items");
      if (items) {
        this.localStorageItems = JSON.parse(items);
      }
    },
    loadCheckboxState() {
      const checkboxState = localStorage.getItem("hasObservationCheckbox");
      if (checkboxState) {
        this.hasObservationCheckbox = JSON.parse(checkboxState);
      }
    },
    saveCheckboxState() {
      localStorage.setItem("hasObservationCheckbox", JSON.stringify(this.hasObservationCheckbox));
    },
    updateInputLength(index) {
      this.inputLengths[index] = this.localStorageItems[index].observation.length;
    },
  },
  mounted() {
    this.loadLocalStorageItems();
    this.loadCheckboxState();
  },
  watch: {
    selectedItemIndex(newIndex, oldIndex) {
      if (oldIndex !== -1) {
        this.localStorageItems[oldIndex].selected = false;
      }
      if (newIndex !== -1) {
        this.localStorageItems[newIndex].selected = true;
      }
    },
  },
};
</script>
