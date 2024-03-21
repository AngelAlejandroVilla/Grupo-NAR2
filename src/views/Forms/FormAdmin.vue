<template>
  <!--begin::Formulario Widget-->
  <div class="container">
    <div class="row">
      <!-- Sidebar -->
      <div class="col-lg-3">
        <div class="card mb-3">
          <div class="card-header">
            <h4 class="card-title mb-0">Formularios Guardados</h4>
          </div>
          <ul class="list-group list-group-flush">
            <li
              v-for="(savedForm, index) in savedForms"
              :key="index"
              class="list-group-item d-flex justify-content-between align-items-center"
            >
              <span>{{ savedForm.title }}</span>
              <div>
                <button
                  @click="loadSavedForm(savedForm)"
                  class="btn btn-primary btn-sm me-2"
                >
                  <i class="bi bi-cloud-download"></i>
                  <!-- Icono de descarga -->
                </button>
                <button
                  @click="removeSavedForm(index)"
                  class="btn btn-danger btn-sm"
                >
                  <i class="bi bi-trash"></i>
                  <!-- Icono de eliminar -->
                </button>
              </div>
            </li>
          </ul>
          <div class="py-3 text-center">
            <button class="btn btn-primary">
              Nuevo Formulario <i class="bi bi-file-earmark-plus"></i>
            </button>
          </div>
        </div>
      </div>
      <!-- /Sidebar -->
      <div class="col-lg-9">
        <div class="card">
          <div class="card-body py-3">
            <form @submit.prevent="submitForm">
              <div class="mb-3">
                <input
                  type="text"
                  v-model="formTitle"
                  class="form-control form-title"
                  placeholder="Título del formulario"
                />
                <textarea
                  v-model="formDesc"
                  class="form-control form-title"
                  placeholder="Descripción del formulario"
                ></textarea>
              </div>
              <div
                v-for="(question, index) in questions"
                :key="index"
                class="mb-3"
              >
                <div class="row">
                  <div class="col-md-8 col-12">
                    <label :for="'question-' + index" class="form-label"
                      >Pregunta {{ index + 1 }}:</label
                    >
                    <input
                      type="text"
                      v-model="question.pregunta"
                      class="form-control form-control-lg form-control-solid mb-3"
                      placeholder="Ingrese una pregunta"
                    />
                  </div>
                  <div class="col-md-3">
                    <div>
                      <label for="tipoPregunta" class="form-label"
                        >Tipo de Pregunta</label
                      >
                      <select
                        v-model="questions[index].tipo"
                        @change="updateQuestionType(index, $event.target.value)"
                        class="form-control form-select"
                      >
                        <option value="text">Texto</option>
                        <option value="radio">Opción múltiple</option>
                      </select>
                    </div>
                  </div>
                  <div class="col-md-1">
                    <div class="mt-7">
                      <button
                        type="button"
                        @click="removeQuestion(index)"
                        class="btn btn-remove"
                      >
                        <i class="bi bi-trash fs-1 fw-bold"></i>
                      </button>
                    </div>
                  </div>
                </div>
                <div class="row">
                  <div v-if="question.tipo === 'radio' && question.opciones">
                    <div
                      v-for="(opcion, opcIndex) in question.opciones"
                      :key="opcIndex"
                    >
                      <div
                        class="input-group"
                        style="margin-top: 12px; margin-bottom: 12px"
                      >
                        <input
                          type="text"
                          :id="'opcion-' + index + '-' + opcIndex"
                          :value="opcion.valor"
                          v-model="question.opciones[opcIndex].valor"
                          class="form-control"
                          placeholder="Opción"
                        />
                        <button
                          type="button"
                          @click="removeOption(index, opcIndex)"
                          class="btn btn-outline-danger"
                        >
                          <i class="bi bi-x fs-2 fw-700"></i>
                        </button>
                      </div>
                    </div>
                    <span
                      @click="addOption(index)"
                      style="cursor: pointer; padding: 12px"
                    >
                      <i
                        style="font-size: 1.5rem"
                        class="bi bi-plus-circle"
                      ></i>
                      <span class="text-muted fs-3">Agregar Opcion</span>
                    </span>
                  </div>
                </div>
              </div>
              <button
                type="button"
                @click="addQuestion"
                class="btn btn-primary me-2"
              >
                Agregar Pregunta
                <i class="bi bi-plus-lg"></i>
              </button>
              <button type="submit" class="btn btn-primary ms-2">
                Guardar Formulario
                <i class="bi bi-floppy"></i>
              </button>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { ref, defineComponent } from "vue";

export default defineComponent({
  setup() {
    const questions = ref([]);
    const formTitle = ref<string>("");
    const formDesc = ref<string>("");
    const savedForms = ref([]);
    const selectedType = ref<string>("text");

    questions.value = [
      {
        tipo: "text",
        pregunta: "",
        opciones: [],
      },
    ];

    const addQuestion = () => {
      let newQuestion = {
        tipo: selectedType.value,
        pregunta: "",
        opciones: [],
      };
      if (selectedType.value === "radio") {
        newQuestion.opciones = [];
      }
      questions.value.push(newQuestion);
    };

    const removeQuestion = (index) => {
      questions.value.splice(index, 1);
    };

    const addOption = (index) => {
      questions.value[index].opciones.push({ nombre: "", valor: "" });
    };

    const removeOption = (qIndex, oIndex) => {
      questions.value[qIndex].opciones.splice(oIndex, 1);
    };

    const updateQuestionType = (index, newType) => {
      questions.value[index].tipo = newType;
      if (newType === "radio") {
        questions.value[index].opciones = []; // Reiniciar opciones si es pregunta de opción múltiple
      }
    };
    const submitForm = () => {
      // Aquí puedes enviar el formulario o hacer lo que necesites con las preguntas
      console.log("Formulario enviado:", {
        title: formTitle.value,
        questions: questions.value,
        description: formDesc.value,
      });

      // Guardar el formulario
      savedForms.value.push({
        title: formTitle.value,
        questions: [...questions.value],
        description: formDesc.value,
      });

      // Limpiar el formulario después de guardar
      clearForm();
    };
    const clearForm = () => {
      formTitle.value = "";
      formDesc.value = "";
      questions.value = [];
    };
    const loadSavedForm = (savedForm) => {
      formTitle.value = savedForm.title;
      formDesc.value = savedForm.description;
      questions.value = [...savedForm.questions];
    };
    const removeSavedForm = (index) => {
      savedForms.value.splice(index, 1);
    };

    return {
      addQuestion,
      removeQuestion,
      submitForm,
      questions,
      formTitle,
      formDesc,
      loadSavedForm,
      savedForms,
      removeSavedForm,
      addOption,
      removeOption,
      updateQuestionType,
    };
  },
});
</script>

<style scoped>
.form-container {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
.form-select {
  display: inline;
  width: 100%;
  padding: 10px;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #fff;
  transition: border-color 0.3s ease-in-out;
}
.form-group {
  margin-bottom: 20px;
  position: relative;
}

.form-label {
  font-weight: bold;
  margin-bottom: 8px;
  display: inline-block;
  color: #333;
}

.form-input {
  width: 100%;
  padding: 12px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  transition: border-color 0.3s;
}

.form-input:focus {
  outline: none;
  border-color: #66afe9;
}

.form-title {
  border: none;
  outline: none;
  background-color: transparent;
  font-size: 20px;
  margin-bottom: 12px;
}
</style>
