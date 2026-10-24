<script setup lang="ts">
import { computed } from 'vue';

interface Student {
    id: number;
    name: string;
    course: string;
    grade: number;
}

const props = defineProps<{ student: Student }>();
const emit = defineEmits<{ (e: 'remove', id: number): void }>();

const passed = computed(() => props.student.grade >= 75);
</script>

<template>
    <tr class="border-b">
        <td class="p-2">{{ student.name }}</td>
        <td class="p-2">{{ student.course }}</td>
        <td
            class="p-2 font-semibold"
            :class="passed ? 'text-green-600' : 'text-red-600'"
        >
            {{ student.grade }}
        </td>
        <td class="p-2">
            <div class="h-2 w-32 rounded bg-gray-200">
                <div
                    class="h-2 rounded"
                    :class="passed ? 'bg-green-500' : 'bg-red-500'"
                    :style="{ width: student.grade + '%' }"
                ></div>
            </div>
        </td>
        <td class="p-2">{{ passed ? 'Passed' : 'Failed' }}</td>
        <td class="p-2">
            <button class="text-red-600 underline" @click="emit('remove', student.id)">
                Delete
            </button>
        </td>
    </tr>
</template>
