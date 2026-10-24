<script setup lang="ts">
import { ref, reactive, computed, watch } from 'vue';
import StudentRow from '@/components/StudentRow.vue';

interface Student {
    id: number;
    name: string;
    course: string;
    grade: number;
}

// ref() + arrays + objects
const students = ref<Student[]>([]);
const showTable = ref(true);
const message = ref('');
let nextId = 1;

// Reactive data (form object)
const form = reactive({ name: '', course: '', grade: '' });
const errors = reactive({ name: '', course: '', grade: '' });

// Form validation
function validate(): boolean {
    const g = Number(form.grade);
    errors.name = form.name.trim() ? '' : 'Name is required.';
    errors.course = form.course.trim() ? '' : 'Course is required.';
    errors.grade =
        form.grade === '' || isNaN(g) || g < 0 || g > 100
            ? 'Grade must be a number from 0 to 100.'
            : '';
    return !errors.name && !errors.course && !errors.grade;
}

// Event methods
function addStudent() {
    if (!validate()) return;
    students.value.push({
        id: nextId++,
        name: form.name.trim(),
        course: form.course.trim(),
        grade: Number(form.grade),
    });
    form.name = '';
    form.course = '';
    form.grade = '';
}

function removeStudent(id: number) {
    students.value = students.value.filter((s) => s.id !== id);
}

function clearAll() {
    students.value = [];
}

// computed() / reactive calculations
const total = computed(() => students.value.length);

const average = computed(() => {
    if (total.value === 0) return 0;
    const sum = students.value.reduce((acc, s) => acc + s.grade, 0);
    return Math.round((sum / total.value) * 100) / 100;
});

const highest = computed(() =>
    total.value ? Math.max(...students.value.map((s) => s.grade)) : 0,
);

const passedCount = computed(() => students.value.filter((s) => s.grade >= 75).length);

// Watcher
watch(total, (newVal, oldVal) => {
    message.value =
        newVal > oldVal ? 'Student added.' : newVal === 0 ? 'All records cleared.' : 'Student removed.';
    setTimeout(() => (message.value = ''), 2000);
});
</script>

<template>
    <div class="mx-auto max-w-3xl p-6">
        <h1 class="mb-4 text-2xl font-bold">Student Grade Tracker</h1>

        <!-- 1. Data entry form -->
        <form class="mb-6 grid gap-3 rounded border p-4" @submit.prevent="addStudent">
            <div>
                <input v-model="form.name" placeholder="Student name" class="w-full rounded border p-2 text-black" />
                <p v-if="errors.name" class="text-sm text-red-600">{{ errors.name }}</p>
            </div>
            <div>
                <input v-model="form.course" placeholder="Course" class="w-full rounded border p-2 text-black" />
                <p v-if="errors.course" class="text-sm text-red-600">{{ errors.course }}</p>
            </div>
            <div>
                <input v-model="form.grade" placeholder="Grade (0-100)" class="w-full rounded border p-2 text-black" />
                <p v-if="errors.grade" class="text-sm text-red-600">{{ errors.grade }}</p>
            </div>
            <button type="submit" class="rounded bg-blue-600 p-2 text-white">Add Student</button>
        </form>

        <!-- Watcher message -->
        <p v-if="message" class="mb-2 text-sm text-blue-600">{{ message }}</p>

        <!-- 7. Computed results -->
        <div class="mb-4 grid grid-cols-4 gap-2 text-center">
            <div class="rounded border p-2">Total<br /><b>{{ total }}</b></div>
            <div class="rounded border p-2">Average<br /><b>{{ average }}</b></div>
            <div class="rounded border p-2">Highest<br /><b>{{ highest }}</b></div>
            <div class="rounded border p-2">Passed<br /><b>{{ passedCount }}</b></div>
        </div>

        <!-- 3. User interaction -->
        <div class="mb-2 flex gap-3">
            <button class="rounded border px-3 py-1" @click="showTable = !showTable">
                {{ showTable ? 'Hide' : 'Show' }} Table
            </button>
            <button v-if="total > 0" class="rounded border px-3 py-1 text-red-600" @click="clearAll">
                Clear All
            </button>
        </div>

        <!-- 4. Conditional rendering + 2/5. Data display & list rendering -->
        <div v-show="showTable">
            <table v-if="total > 0" class="w-full text-left">
                <thead>
                    <tr class="border-b">
                        <th class="p-2">Name</th>
                        <th class="p-2">Course</th>
                        <th class="p-2">Grade</th>
                        <th class="p-2">Progress</th>
                        <th class="p-2">Status</th>
                        <th class="p-2"></th>
                    </tr>
                </thead>
                <tbody>
                    <StudentRow
                        v-for="student in students"
                        :key="student.id"
                        :student="student"
                        @remove="removeStudent"
                    />
                </tbody>
            </table>
            <p v-else class="py-4 text-gray-500">No records yet. Add a student above.</p>
        </div>
    </div>
</template>
