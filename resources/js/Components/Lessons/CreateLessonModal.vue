<script setup>
import { ref, watch } from 'vue';
import Dialog from 'primevue/dialog';
import Button from 'primevue/button';
import InputText from 'primevue/inputtext';
import Textarea from 'primevue/textarea';
import Dropdown from 'primevue/dropdown';
import MultiSelect from 'primevue/multiselect';
import InputGroup from 'primevue/inputgroup';
import InputGroupAddon from 'primevue/inputgroupaddon';
import { useToast } from "primevue/usetoast";
import { getFileTypeIcon, getFileTypeColor } from '@/Utils/fileHelpers';

const props = defineProps({
    visible: Boolean,
    grades: Array,
});

const emit = defineEmits(['update:visible', 'save']);

const lesson = ref({
    youtubeUrl: '',
    title: '',
    description: '',
    grade: null,
    tags: [],
    resources: [] // Now will store drive links
});

const loading = ref(false);
const availableTags = ref([
    { name: 'Mathematics', code: 'math' },
    { name: 'Science', code: 'science' },
    { name: 'English', code: 'english' }
]);

// New tag creation
const newTag = ref('');
const handleNewTag = () => {
    const value = newTag.value.trim();
    if (value) {
        // Add tag if not exists (case insensitive)
        if (!availableTags.value.find(tag => tag.name.toLowerCase() === value.toLowerCase())) {
            const tag = { code: value.toLowerCase().replace(/\s+/g, '-'), name: value };
            availableTags.value.push(tag);
            lesson.value.tags.push(tag);
        }
        newTag.value = '';
    }
};

const fetchYoutubeData = () => {
    loading.value = true;
    // Simulate API call
    setTimeout(() => {
        lesson.value.title = 'Sample YouTube Video Title';
        lesson.value.description = 'This is a sample description fetched from YouTube API';
        loading.value = false;
    }, 1000);
};

const createLesson = () => {
    emit('save', lesson.value);
    emit('update:visible', false);
};

const toast = useToast();

const handleUploadSuccess = (message) => {
    toast.add({
        severity: 'success',
        summary: 'Success',
        detail: message,
        life: 3000
    });
};

const handleUploadError = (error) => {
    toast.add({
        severity: 'error',
        summary: 'Error',
        detail: error,
        life: 3000
    });
};

// New resource handling
const newResource = ref({
    name: '',
    url: '',
    type: 'pdf' // Default type
});

const resourceTypes = [
    { label: 'PDF Document', value: 'pdf' },
    { label: 'Word Document', value: 'doc' },
    { label: 'PowerPoint', value: 'ppt' },
    { label: 'Excel Sheet', value: 'xlsx' },
    { label: 'Other', value: 'other' }
];

const addResource = () => {
    if (newResource.value.name && newResource.value.url) {
        lesson.value.resources.push({
            name: newResource.value.name,
            url: newResource.value.url,
            type: newResource.value.type
        });
        // Reset form
        newResource.value = {
            name: '',
            url: '',
            type: 'pdf'
        };
    }
};

const removeResource = (index) => {
    lesson.value.resources.splice(index, 1);
};
</script>

<template>
    <Dialog 
        :visible="props.visible" 
        modal 
        header="Create New Lesson" 
        :style="{ width: '50vw' }"
        @update:visible="$emit('update:visible', $event)"
    >
        <div class="space-y-4">
            <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">YouTube URL</label>
                <InputGroup>
                    <InputText v-model="lesson.youtubeUrl" placeholder="Enter YouTube URL" class="w-full" />
                    <InputGroupAddon>
                        <Button 
                            icon="pi pi-search" 
                            @click="fetchYoutubeData" 
                            :loading="loading"
                        />
                    </InputGroupAddon>
                </InputGroup>
            </div>

            <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">Title</label>
                <InputText v-model="lesson.title" placeholder="Lesson title" class="w-full" />
            </div>

            <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">Description</label>
                <Textarea v-model="lesson.description" rows="3" class="w-full" />
            </div>

            <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">Grade</label>
                <Dropdown 
                    v-model="lesson.grade" 
                    :options="grades" 
                    placeholder="Select Grade"
                    class="w-full"
                />
            </div>

            <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">Tags</label>
                <MultiSelect
                    v-model="lesson.tags"
                    :options="availableTags"
                    optionLabel="name"
                    placeholder="Select Tags"
                    :filter="true"
                    class="w-full"
                />
                <InputGroup class="mt-2">
                    <InputGroupAddon>
                        <i class="pi pi-plus" />
                    </InputGroupAddon>
                    <InputText
                        v-model="newTag"
                        placeholder="Type tag then press Enter"
                        @keydown.enter.prevent="handleNewTag"
                        class="w-full"
                    />
                </InputGroup>
            </div>

            <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">
                    Lesson Resources
                </label>
                
                <!-- Add new resource form -->
                <div class="border border-gray-200 hover:border-primary-500 transition-colors duration-200 rounded-lg p-4 space-y-3 mb-4 shadow-sm">
                    <div>
                        <label class="text-sm text-gray-600">Resource Name</label>
                        <InputText 
                            v-model="newResource.name"
                            placeholder="Enter resource name"
                            class="w-full"
                        />
                    </div>
                    
                    <div>
                        <label class="text-sm text-gray-600">Drive Link</label>
                        <InputGroup>
                            <InputGroupAddon>
                                <i class="pi pi-link" />
                            </InputGroupAddon>
                            <InputText 
                                v-model="newResource.url"
                                placeholder="Paste Google Drive link"
                                class="w-full"
                            />
                        </InputGroup>
                    </div>

                    <div>
                        <label class="text-sm text-gray-600">Resource Type</label>
                        <Dropdown
                            v-model="newResource.type"
                            :options="resourceTypes"
                            optionLabel="label"
                            optionValue="value"
                            class="w-full"
                        />
                    </div>

                    <Button 
                        label="Add Resource" 
                        icon="pi pi-plus"
                        @click="addResource"
                        :disabled="!newResource.name || !newResource.url"
                        class="w-full"
                    />
                </div>

                <!-- Resource List -->
                <div v-if="lesson.resources.length > 0" class="space-y-2 border border-gray-100 rounded-lg p-3">
                    <div v-for="(resource, index) in lesson.resources" 
                         :key="index"
                         class="flex items-center justify-between p-3 bg-gray-50 rounded-lg border border-gray-100 hover:border-primary-500 hover:bg-primary-50 transition-all duration-200">
                        <div class="flex items-center gap-3">
                            <i :class="[getFileTypeIcon(resource.type), getFileTypeColor(resource.type), 'text-xl']" />
                            <div class="flex flex-col">
                                <span class="text-sm font-medium">{{ resource.name }}</span>
                                <a :href="resource.url" 
                                   target="_blank" 
                                   class="text-xs text-blue-600 hover:underline">
                                    Open in Drive
                                </a>
                            </div>
                        </div>
                        <Button 
                            icon="pi pi-times" 
                            text
                            severity="danger"
                            @click="removeResource(index)"
                        />
                    </div>
                </div>
            </div>
        </div>

        <template #footer>
            <Button label="Cancel" @click="$emit('update:visible', false)" text />
            <Button label="Create Lesson" @click="createLesson" />
        </template>
    </Dialog>
</template>
