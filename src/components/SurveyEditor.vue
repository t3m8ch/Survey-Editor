<template>
  <ul>
    <li v-for="q in survey" :key="q.id">
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
  </ul>
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

    return { addQuestion, deleteQuestion, onInputAnswer };
  },
};
</script>

<style scoped></style>
