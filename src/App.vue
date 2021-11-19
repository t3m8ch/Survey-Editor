<template>
  <survey-editor
    :survey="survey"
    @updateSurvey="onUpdateSurvey"
    @addQuestion="onAddQuestion"
    @deleteQuestion="onDeleteQuestion"
    @addAnswer="onAddAnswer"
    @deleteAnswers="onDeleteAnswers" />
</template>

<script>
import SurveyEditor from "./components/SurveyEditor";
import { ref, watch } from "vue";

export default {
  name: "App",
  components: { SurveyEditor },
  setup() {
    const survey = ref([]);

    watch(survey, (newSurvey) => console.log(newSurvey), { deep: true });

    const onUpdateSurvey = (newSurvey) => survey.value = newSurvey;

    const onAddQuestion = (newSurvey) => survey.value.push(newSurvey);
    const onDeleteQuestion = (questionId) => {
      survey.value = survey.value.filter((q) => q.id !== questionId);
    };

    const onAddAnswer = ({ questionId, newAnswer }) => {
      const question = survey.value.find((q) => q.id === questionId);
      question.answers.push(newAnswer);
    };
    const onDeleteAnswers = ({ questionId, answersIds }) => {
      const question = survey.value.find((q) => q.id === questionId);
      question.answers = question.answers.filter(
        (a) => !answersIds.includes(a.id)
      );
    };

    return {
      survey,
      onUpdateSurvey,
      onAddQuestion,
      onDeleteQuestion,
      onAddAnswer,
      onDeleteAnswers,
    };
  },
};
</script>
