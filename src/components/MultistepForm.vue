<script setup>
import { ref, computed } from "vue";
import formSchema from "@/data/formSchema.json";
import Swal from "sweetalert2";


const currentStepIndex = ref(0);
const submitted = ref(false);
const formData = ref({});
const steps = formSchema;

const currentStep = computed(() => steps[currentStepIndex.value]);

const nextStep = () => {
  if (currentStepIndex.value < steps.length - 1) {
    currentStepIndex.value++;
  }
};

const prevStep = () => {
  if (currentStepIndex.value > 0) {
    currentStepIndex.value--;
  }
};

const handleSubmit = () => {
  const requiredFields = currentStep.value.fields.filter(
    (field) => field.required
  );
  let allFieldsFilled = true;

  requiredFields.forEach((field) => {
    if (!formData.value[field.label]) {
      allFieldsFilled = false;
    }
  });

  if (!allFieldsFilled) {
    Swal.fire({
      icon: "error",
      title: "Oops...",
      text: "Please fill out all required fields!",
      toast: true,
      position: "top",
      showConfirmButton: false,
      timer: 3000,
      timerProgressBar: true,
      customClass: {
        popup: "swal-toast-popup",
      },
    });
  } else {
    if (currentStepIndex.value < steps.length - 1) {
      nextStep();
    } else {
      Swal.fire({
        icon: "success",
        title: "Success!",
        text: "Form submitted successfully!",
        toast: true,
        position: "top",
        showConfirmButton: false,
        timer: 3000,
        timerProgressBar: true,
        customClass: {
          popup: "swal-toast-popup",
        },
      })
      submitted.value = true;

      console.log("Form submitted:", formData.value);
    }
  }
};




const resetForm = () => {
  submitted.value = false;
  formData.value = {};
  currentStepIndex.value = 0;
};
</script>

<template>
  <div class="flex h-screen w-full justify-center items-center">
    <div class="bg-white p-6 w-full h-full md:rounded-lg md:max-w-md md:h-fit md:shadow-lg">
      <!-- Progress bar -->
      <div class="flex justify-between my-6" >
        <!-- Step 1 Indicator -->
        <div
          class="flex flex-col items-center justify-center w-6 h-6 rounded-full transition-colors duration-1000 ease-in-out cursor-pointer" @click="prevStep" 
          :class="{
            'bg-emerald-300': currentStep.step === 1,
            'bg-gray-300': currentStep.step !== 1,
          }"
        >
          <p class="font-bold text-xs p-2 text-white">1</p>
        </div>

        <!-- Progress Bar -->
        <div class="h-1 bg-gray-300 w-5/6 my-auto mx-2">
          <div
            class="bg-emerald-300 h-1 rounded-full transition-all duration-500 ease-in-out"
            :style="{ width: currentStep.step === 2 ? '100%' : '50%' }"
          ></div>
        </div>

        <!-- Step 2 Indicator -->
        <div
          class="flex flex-col items-center justify-center w-6 h-6 rounded-full transition-colors duration-1000 ease-in-out cursor-pointer"
          :class="{
            'bg-emerald-300': currentStep.step === 2,
            'bg-gray-300': currentStep.step !== 2,
          }"
        >
          <p class="font-bold text-xs p-2 text-white">2</p>
        </div>
      </div>

      <!-- Form Title and Description -->
      <div v-if="!submitted" class="mb-4">
        <h1 class="text-2xl font-bold text-center">{{ currentStep.title }}</h1>
        <p class="mb-4 text-sm text-center">{{ currentStep.description }}</p>
      </div>

      <!-- Form Fields -->
      <form v-if="!submitted" @submit.prevent="handleSubmit">
        <div
          v-for="(field, index) in currentStep.fields"
          :key="index"
          class="mb-4 mt-6"
        >
          <label class="block mb-1 text-sm">{{ field.label }}</label>

          <!-- Text Field -->
          <template v-if="field.type === 'textfield'">
            <input
              type="text"
              v-model="formData[field.label]"
              :placeholder="field.placeholder"
              :required="field.required"
              class="border-gray-300 text-sm p-2 rounded-lg w-full focus:border-emerald-300 focus:outline-none border-2"
            />
          </template>

          <!-- Text Area -->
          <template v-if="field.type === 'textarea'">
            <textarea
              v-model="formData[field.label]"
              :placeholder="field.placeholder"
              :required="field.required"
              class="border-gray-300 text-sm p-2 rounded-lg focus:border-emerald-300 focus:outline-none border-2 w-full"
            ></textarea>
          </template>

          <!-- Radio Buttons -->
          <template v-if="field.type === 'radio'">
            <div>
              <template v-for="option in field.options" :key="option.value">
                <label class="mr-4 text-sm">
                  <input
                    type="radio"
                    v-model="formData[field.label]"
                    :value="option.value"
                    :required="field.required"
                    required="true"
                  />
                  {{ option.label }}
                </label>
              </template>
            </div>
          </template>

          <!-- Autocomplete -->
          <template v-if="field.type === 'autocomplete'">
            <input
              type="text"
              v-model="formData[field.label]"
              :placeholder="field.placeholder"
              :required="field.required"
              list="options"
              class="border-2 border-gray-300 p-2 rounded-lg text-sm focus:border-emerald-300 focus:outline-none w-full"
            />
            <datalist id="options">
              <option
                v-for="option in field.options"
                :key="option"
                :value="option"
              >
                {{ option }}
              </option>
            </datalist>
          </template>
        </div>

        <!-- Navigation Buttons -->
        <div class="flex justify-between mt-6">
          <button
            type="button"
            @click="prevStep"
            class="bg-gray-300 text-white px-4 font-bold text-sm py-2 rounded hover:bg-gray-500 transition-colors duration-300 ease-in-out"  
            :disabled="currentStepIndex === 0"
          >
            Previous
          </button>

          <button
            type="submit"
            class="bg-emerald-300 font-bold text-sm text-white px-4 py-2 rounded hover:bg-emerald-500 transition-colors duration-300 ease-in-out"
          >
            {{
              currentStepIndex === steps.length - 1 ? "Submit" : "Next"
            }}
          </button>
        </div>
      </form>

      <!-- Submission Result -->
      <div v-if="submitted">
        <h1 class="text-2xl font-bold text-center">Result Form</h1>
        <p class="mb-4 text-sm text-center">
          Here is the form you've submitted
        </p>
        <div class="border border-gray-300 p-4 rounded-lg space-y-2">
          <p>Title : {{ formData.Title }}</p>
          <p>Name : {{ formData.Name }}</p>
          <p>Gender : {{ formData.Gender }}</p>
          <p>Description : {{ formData.Description }}</p>
        </div>

        <button
          @click="resetForm"
          class="mt-4 bg-emerald-300 text-white font-bold text-sm px-4 py-2 rounded hover:bg-emerald-500 transition-colors duration-300 ease-in-out"
        >
          Go Back
        </button>
      </div>
    </div>
  </div>
</template>



<style scoped></style>
