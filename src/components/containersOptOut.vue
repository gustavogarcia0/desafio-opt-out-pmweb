<template>
  <div>
    <!-- Container 1 -->
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

    <!-- Container 2 -->
    <div class="container card2" v-if="showContainer2 && justificativas.length > 0">
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
          <li class="itemJustificativa" v-for="(justificativa, index) in justificativas" :key="justificativa.id">
            <div>
              <input
                type="radio"
                :value="index"
                :id="'radio-' + index"
                :checked="selectedItemIndex === index"
                @change="selectedItemIndex = index" />
              <label :for="'radio-' + index">{{ justificativa.text }}</label>
              <button class="excluir-justificativa" @click="deleteJustificativa(justificativa.id, index)">Excluir</button>
            </div>
            <div class="input-text-justificativa">
              <input
                id="justificativa"
                type="text"
                v-if="justificativa.tem_observacao && selectedItemIndex === index"
                v-model="observacoes[index]"
                maxlength="150" />
              <span class="input_length" v-if="justificativa.tem_observacao && selectedItemIndex === index">
                <span>{{ observacoes[index] ? observacoes[index].length : 0 }}</span
                >/150 caracteres
              </span>
            </div>
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
import axios from "axios";
import "./styles/container1.css";
import "./styles/container2.css";
import "./styles/global.css";

export default {
  data() {
    return {
      inputText: "", // Texto inserido no campo de justificativa
      hasObservationCheckbox: false, // Indica se o usuário deseja informar uma observação
      justificativas: [], // Lista de justificativas disponíveis
      selectedItemIndex: -1, // Índice da justificativa selecionada
      inputLengths: [], // Comprimento do texto inserido em cada justificativa
      showContainer2: false, // Indica se o Container 2 deve ser exibido
      observacoes: [], // Lista de observações informadas pelo usuário
    };
  },
  methods: {
    /**
     * Envia a justificativa do Container 1 para a API.
     * Atualiza a lista de justificativas com a nova justificativa criada.
     * Limpa o campo de texto e exibe o Container 2.
     */
    async submitContainer1() {
      const texto = this.inputText;
      const temObservacao = this.hasObservationCheckbox;
      const response = await axios.post("https://node-api-dev-challenge.onrender.com/justificativas", {
        texto: texto,
        tem_observacao: temObservacao,
      });
      const justificativa = response.data;
      justificativa.observacao = "";
      this.justificativas.push(justificativa);
      this.inputText = "";
      this.showContainer2 = true;
      await this.fetchJustificativas();
    },

    /**
     * Remove uma justificativa da lista de justificativas.
     * Verifica se o Container 2 deve ser ocultado.
     *
     * @param {number} id - ID da justificativa a ser excluída.
     */
    async deleteJustificativa(id) {
      try {
        await axios.delete(`https://node-api-dev-challenge.onrender.com/justificativas/${id}`);
        const index = this.justificativas.findIndex((justificativa) => justificativa.id === id);
        if (index !== -1) {
          this.justificativas.splice(index, 1);
        }
      } catch (error) {
        console.error(error);
      }
      if (this.justificativas.length === 0) {
        this.showContainer2 = false;
      }
    },

    /**
     * Envia as justificativas do Container 2 para a API.
     * Exibe as justificativas no console.
     */
    async submitContainer2() {
      console.log(this.justificativas);
    },

    /**
     * Atualiza o comprimento do texto inserido em uma justificativa.
     *
     * @param {number} index - Índice da justificativa.
     */
    updateInputLength(index) {
      this.inputLengths[index] = this.justificativas[index].observacao.length;
    },

    /**
     * Busca as justificativas da API e atualiza a lista de justificativas.
     */
    async fetchJustificativas() {
      try {
        const response = await axios.get("https://node-api-dev-challenge.onrender.com/justificativas");
        this.justificativas = response.data.map((justificativa) => {
          return {
            id: justificativa.id,
            text: justificativa.texto,
            tem_observacao: justificativa.tem_observacao,
            observacao: "",
            selected: false,
          };
        });
        this.inputLengths = this.justificativas.map(() => 0);
        if (this.justificativas.length > 0) {
          this.showContainer2 = true;
        }
      } catch (error) {
        console.error(error);
      }
    },
  },

  mounted() {
    this.fetchJustificativas();
  },

  watch: {
    /**
     * Atualiza o estado selecionado das justificativas quando o selectedItemIndex é alterado.
     *
     * @param {number} newIndex - Novo índice da justificativa selecionada.
     * @param {number} oldIndex - Antigo índice da justificativa selecionada.
     */
    selectedItemIndex(newIndex, oldIndex) {
      if (oldIndex !== -1) {
        this.justificativas[oldIndex].selected = false;
      }
      if (newIndex !== -1) {
        this.justificativas[newIndex].selected = true;
      }
    },
  },
};
</script>
