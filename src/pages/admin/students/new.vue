<template>
  <DStudentForm :student="student" :title="$t('add_student')" @save="onCreateStudent"></DStudentForm>
</template>

<script lang="ts" setup>
import DStudentForm from "./DStudentForm.vue";
import { User } from "@/gql/graphql";
import { reactive } from "vue";
import { useMutation } from "@urql/vue";
import { graphql } from "@/gql";
import { createNotification } from "@/composables/useToast";
import { useRouter } from "vue-router";

const router = useRouter();

const student = reactive<User>({
  firstName: "",
  lastName: "",
  // @ts-expect-error
  student: {
    grade: 0,
    birthday: null,
    leftAt: null,
    joinedAt: null,
  },
});

const { executeMutation: createStudent } = useMutation(
  graphql(`
    mutation createStudent($student: CreateStudentInput!) {
      createStudent(input: $student) {
        id
        firstName
        lastName
        student {
          id
          birthday
          grade
          leftAt
          joinedAt
        }
      }
    }
  `)
);

const onCreateStudent = async () => {
  if (!student.firstName) {
    alert("First name is required");
    return;
  }
  if (!student.lastName) {
    alert("Last name is required");
    return;
  }
  if (!student.student?.grade) {
    alert("Grade is required");
    return;
  }
  if (student.student.grade < 0 || student.student.grade > 13) {
    alert("Grade must be between 0 and 13");
    return;
  }

  await createStudent({
    student: {
      firstName: student.firstName,
      lastName: student.lastName,
      grade: student.student?.grade,
      birthday: student.student?.birthday,
      leftAt: student.student?.leftAt,
      joinedAt: student.student?.joinedAt,
    },
  });

  await router.push({ name: "admin-students" });

  await createNotification({
    title: "Student created",
    description: `${student.firstName} ${student.lastName} was created`,
  });
};
</script>
