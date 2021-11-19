<template>
  <transition-group name="flip-list" tag="ul">
    <li v-for="(q, index) in survey" :key="q.id" class="question">
      <div style="display: flex">
        <button
          @click="moveQuestionDown(q.id)"
          :disabled="index === survey.length - 1">
          Вниз
        </button>
        <button @click="moveQuestionUp(q.id)" :disabled="index === 0">
          Вверх
        </button>
      </div>
      <div style="display: flex">
        <h3>{{ q.question }}</h3>
        <button style="margin-left: 1rem" @click="deleteQuestion(q.id)">
          X
        </button>
      </div>
      <input type="text" v-model="q.question" />
      <ul>
        <li v-for="answer in q.answers" :key="answer.id">
          <span>{{ answer.value }}</span>
          <input type="text" v-model="answer.value" @input="onInputAnswer" />
        </li>
      </ul>
    </li>
  </transition-group>
  <button @click="addQuestion">Добавить вопрос</button>
</template>

<script>
const getNewId = (item) => Math.max(Math.max(...item.map((q) => q.id)) + 1, 1);

export default {
  name: "SurveyEditor",
  props: {
    survey: {
      type: Array,
      required: true,
    },
  },
  emits: {
    updateSurvey: null,

    addQuestion: null,
    deleteQuestion: (questionId) => typeof questionId === "number",

    addAnswer: null,
    deleteAnswers: null,
  },
  setup(props, { emit }) {
    const addQuestion = () => {
      const id = getNewId(props.survey);
      emit("addQuestion", {
        id,
        question: `Вопрос ${id}`,
        answers: [{ value: "", id: 1 }],
      });
    };

    const deleteQuestion = (questionId) => emit("deleteQuestion", questionId);

    const addEmptyAnswer = () => {
      const questionsWithoutEmptyAnswer = props.survey.filter(
        (q) => !q.answers.map((a) => a.value).includes("")
      );
      questionsWithoutEmptyAnswer.forEach((q) => {
        emit("addAnswer", {
          questionId: q.id,
          newAnswer: { value: "", id: getNewId(q.answers) },
        });
      });
    };

    const deleteUnnecessaryAnswers = () => {
      const questionWithMoreOneEmptyAnswer = props.survey.filter(
        (q) => q.answers.filter((a) => a.value === "").length > 1
      );
      questionWithMoreOneEmptyAnswer.forEach((q) => {
        const emptyAnswers = q.answers.filter((a) => a.value === "");
        const answersToDelete = [...emptyAnswers].splice(
          1,
          emptyAnswers.length
        );
        emit("deleteAnswers", {
          questionId: q.id,
          answersIds: answersToDelete.map((a) => a.id),
        });
      });
    };

    const onInputAnswer = () => {
      addEmptyAnswer();
      deleteUnnecessaryAnswers();
    };

    const moveQuestionDown = (questionId) => {
      const curr = props.survey.findIndex((q) => q.id === questionId);
      const next = curr + 1;

      let newSurvey = [...props.survey];
      [newSurvey[curr], newSurvey[next]] = [newSurvey[next], newSurvey[curr]];

      emit("updateSurvey", newSurvey);
    };

    const moveQuestionUp = (questionId) => {
      const curr = props.survey.findIndex((q) => q.id === questionId);
      const prev = curr - 1;

      let newSurvey = [...props.survey];
      [newSurvey[curr], newSurvey[prev]] = [newSurvey[prev], newSurvey[curr]];

      emit("updateSurvey", newSurvey);
    };

    return {
      addQuestion,
      deleteQuestion,
      onInputAnswer,
      moveQuestionDown,
      moveQuestionUp,
    };
  },
};
</script>

<style scoped>
.question {
  background: #ddd;
}

.question + .question {
  margin-top: 10px;
}

.flip-list-move {
  transition: transform 0.3s cubic-bezier(0.770, 0.000, 0.175, 1.000);
}
</style>
