<template>
  <Formulario @aoSalvarTarefa="salvarTarefa" />
  <div class="lista is-active">
    <Box v-if="semTarefas">
      Você não está muito produtivo hoje
      <span class="has-text-weight-bold">:(</span>
    </Box>
    <div class="field">
      <label class="label">Pesquisa</label>
      <div class="control has-icons-left">
        <input
          class="input"
          type="text"
          placeholder="Pesquisar"
          v-model="filtro"
        />
        <span class="icon is-small is-left">
          <i class="fas fa-search"></i>
        </span>
      </div>
    </div>
    <Tarefa
      v-for="(tarefa, index) in tarefas"
      :tarefa="tarefa"
      :key="index"
      @aoTarefaClicada="selecionarTarefa"
    />
    <div v-if="tarefaSelecionada">
      <Modal :mostrar="tarefaSelecionada != null">
        <template v-slot:cabecalho>
          <p class="modal-card-title">Editando uma tarefa</p>
          <button
            class="delete"
            aria-label="close"
            @click="fecharModal"
          ></button>
        </template>
        <template v-slot:corpo>
          <div class="field">
            <label for="descricaoDaTarefa" class="label"> Descrição </label>
            <input
              type="text"
              class="input"
              id="descricaoDaTarefa"
              v-model="tarefaSelecionada.descricao"
            />
          </div>
        </template>
        <template v-slot:rodape>
          <button class="button is-success" @click="alterarTarefa">
            Salvar Alterações
          </button>
          <button class="button" @click="fecharModal">Cancelar</button>
        </template>
      </Modal>
    </div>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, ref, watchPostEffect } from "vue";
import Formulario from "../components/Formulario.vue";
import Tarefa from "../components/Tarefa.vue";
import Box from "../components/Box.vue";
import Modal from "../components/Modal.vue";
import ITarefa from "../interfaces/ITarefa";
import { useStore } from "@/store";
import {
  ALTERAR_TAREFA,
  CADASTRAR_TAREFA,
  OBTER_PROJETOS,
  OBTER_TAREFAS,
} from "@/store/tipo-acoes";

export default defineComponent({
  name: "App",
  components: {
    Formulario,
    Tarefa,
    Box,
    Modal,
  },
  data() {
    return {
      tarefaSelecionada: null as ITarefa | null,
    };
  },
  methods: {
    salvarTarefa(tarefa: ITarefa): void {
      this.store.dispatch(CADASTRAR_TAREFA, tarefa);
    },
    selecionarTarefa(tarefa: ITarefa) {
      this.tarefaSelecionada = tarefa;
    },
    fecharModal() {
      this.tarefaSelecionada = null;
    },
    alterarTarefa() {
      this.store
        .dispatch(ALTERAR_TAREFA, this.tarefaSelecionada)
        .then(() => this.fecharModal());
    },
  },
  computed: {
    semTarefas(): boolean {
      return this.tarefas.length == 0;
    },
  },
  setup() {
    const store = useStore();
    store.dispatch(OBTER_PROJETOS);
    store.dispatch(OBTER_TAREFAS);

    const filtro = ref("");
    /* const tarefas = computed(() =>
      store.state.tarefas.filter(
        (t) => !filtro.value || t.descricao.includes(filtro.value)
      )
    ); */

    watchPostEffect(() => {
      store.dispatch(OBTER_TAREFAS, filtro.value);
    });

    return {
      tarefas: computed(() => store.state.tarefas),
      store,
      filtro,
    };
  },
});
</script>
