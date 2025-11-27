<template>
  <v-container class="bookmarks-panel flex-grow-1 overflow-y-auto" style="max-height: calc(100vh - 64px);">
    <v-row justify="center">
      <v-col cols="12" lg="10" xl="8">
        <div class="d-flex align-center mb-4">
          <v-icon size="large" class="mr-3">mdi-bookmark</v-icon>
          <h2 class="text-h5 font-weight-bold">Bookmarks</h2>
        </div>

        <v-alert
          v-if="error"
          type="error"
          variant="tonal"
          class="mb-4"
        >
          {{ error }}
        </v-alert>

        <div v-if="loading" class="d-flex justify-center py-8">
          <v-progress-circular indeterminate color="primary" />
        </div>

        <div v-else>
          <v-alert
            v-if="!bookmarks.length"
            type="info"
            variant="tonal"
            class="mb-4"
          >
            You haven't created any bookmarks yet.
          </v-alert>

          <div v-else>
            <div
              v-for="(group, conversationId) in groupedBookmarks"
              :key="conversationId"
              class="bookmark-group mb-4"
            >
              <div class="group-header">
                <v-icon size="small" class="mr-2">mdi-folder-outline</v-icon>
                <span class="conversation-title">{{ group.title }}</span>
              </div>

              <v-list density="compact" class="bookmark-list">
                <v-list-item
                  v-for="bookmark in group.bookmarks"
                  :key="bookmark.id"
                  class="bookmark-item"
                  @click="handleBookmarkClick(bookmark)"
                >
                  <template #prepend>
                    <v-icon size="small" color="primary">mdi-bookmark</v-icon>
                  </template>

                  <template v-if="isEditingBookmark(bookmark)">
                    <div class="bookmark-edit-field">
                      <v-text-field
                        v-model="editingLabel"
                        variant="outlined"
                        density="compact"
                        hide-details
                        autofocus
                        class="bookmark-edit-input"
                        :disabled="isSavingBookmark(bookmark)"
                        @keydown.enter.prevent="saveBookmarkLabel(bookmark)"
                        @keydown.esc.prevent="cancelEditingBookmark"
                      />
                    </div>
                  </template>
                  <v-list-item-title v-else class="bookmark-label">
                    {{ bookmark.label }}
                  </v-list-item-title>

                  <v-list-item-subtitle class="bookmark-date">
                    {{ formatDate(bookmark.createdAt) }}
                  </v-list-item-subtitle>

                  <template #append>
                    <template v-if="isEditingBookmark(bookmark)">
                      <v-btn
                        size="x-small"
                        variant="text"
                        class="bookmark-action-btn mr-1"
                        :loading="isSavingBookmark(bookmark)"
                        :disabled="isSavingBookmark(bookmark)"
                        @click.stop="saveBookmarkLabel(bookmark)"
                      >
                        <v-icon size="small">mdi-check</v-icon>
                        <v-tooltip activator="parent" location="top">Save</v-tooltip>
                      </v-btn>
                      <v-btn
                        size="x-small"
                        variant="text"
                        class="bookmark-action-btn mr-1"
                        :disabled="isSavingBookmark(bookmark)"
                        @click.stop="cancelEditingBookmark"
                      >
                        <v-icon size="small">mdi-close</v-icon>
                        <v-tooltip activator="parent" location="top">Cancel</v-tooltip>
                      </v-btn>
                    </template>
                    <template v-else>
                      <v-btn
                        size="x-small"
                        variant="text"
                        class="bookmark-action-btn mr-1"
                        @click.stop="startEditingBookmark(bookmark)"
                      >
                        <v-icon size="small">mdi-pencil</v-icon>
                        <v-tooltip activator="parent" location="top">Edit label</v-tooltip>
                      </v-btn>
                      <v-btn
                        size="x-small"
                        variant="text"
                        class="bookmark-action-btn mr-1"
                        @click.stop="copyBookmarkLink(bookmark)"
                      >
                        <v-icon size="small">mdi-link</v-icon>
                        <v-tooltip activator="parent" location="top">Copy link</v-tooltip>
                      </v-btn>
                    </template>
                    <v-icon size="small" class="navigate-icon">mdi-chevron-right</v-icon>
                  </template>
                </v-list-item>
              </v-list>
            </div>
          </div>
        </div>
      </v-col>
    </v-row>
    
    <v-snackbar v-model="copiedSnackbar" :timeout="2000" color="success">
      Link copied to clipboard
    </v-snackbar>
  </v-container>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import { api } from '@/services/api';

interface Bookmark {
  id: string;
  conversationId: string;
  messageId: string;
  branchId: string;
  label: string;
  createdAt: string;
  conversationTitle: string;
}

interface BookmarkGroup {
  title: string;
  bookmarks: Bookmark[];
}

const emit = defineEmits<{
  navigate: [bookmark: Bookmark];
}>();

const bookmarks = ref<Bookmark[]>([]);
const loading = ref(true);
const error = ref('');
const editingBookmarkId = ref('');
const editingLabel = ref('');
const savingBookmarkId = ref('');

const groupedBookmarks = computed(() => {
  const groups: Record<string, BookmarkGroup> = {};

  for (const bookmark of bookmarks.value) {
    if (!groups[bookmark.conversationId]) {
      groups[bookmark.conversationId] = {
        title: bookmark.conversationTitle,
        bookmarks: [],
      };
    }
    groups[bookmark.conversationId].bookmarks.push(bookmark);
  }

  return groups;
});

function formatDate(dateStr: string): string {
  const date = new Date(dateStr);
  return date.toLocaleDateString(undefined, {
    year: 'numeric',
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
  });
}

function handleBookmarkClick(bookmark: Bookmark) {
  if (editingBookmarkId.value === bookmark.id) {
    return;
  }
  emit('navigate', bookmark);
}

const copiedSnackbar = ref(false);

function copyBookmarkLink(bookmark: Bookmark) {
  const url = `${window.location.origin}/conversation/${bookmark.conversationId}?messageId=${bookmark.messageId}&branchId=${bookmark.branchId}`;
  navigator.clipboard.writeText(url);
  copiedSnackbar.value = true;
}

function startEditingBookmark(bookmark: Bookmark) {
  editingBookmarkId.value = bookmark.id;
  editingLabel.value = bookmark.label;
  error.value = '';
}

function cancelEditingBookmark() {
  editingBookmarkId.value = '';
  editingLabel.value = '';
}

function isEditingBookmark(bookmark: Bookmark): boolean {
  return editingBookmarkId.value === bookmark.id;
}

function isSavingBookmark(bookmark: Bookmark): boolean {
  return savingBookmarkId.value === bookmark.id;
}

async function saveBookmarkLabel(bookmark: Bookmark) {
  if (!isEditingBookmark(bookmark)) {
    return;
  }
  const label = editingLabel.value.trim();
  if (!label) {
    return;
  }
  if (label === bookmark.label) {
    cancelEditingBookmark();
    return;
  }
  try {
    savingBookmarkId.value = bookmark.id;
    error.value = '';
    await api.post('/bookmarks', {
      conversationId: bookmark.conversationId,
      messageId: bookmark.messageId,
      branchId: bookmark.branchId,
      label,
    });
    bookmark.label = label;
    cancelEditingBookmark();
  } catch (err) {
    console.error('Failed to update bookmark label', err);
    error.value = 'Unable to update bookmark.';
  } finally {
    savingBookmarkId.value = '';
  }
}

async function loadBookmarks() {
  loading.value = true;
  error.value = '';
  try {
    const response = await api.get('/bookmarks');
    bookmarks.value = response.data || [];
  } catch (err) {
    console.error('Failed to load bookmarks', err);
    error.value = 'Unable to load bookmarks.';
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  loadBookmarks();
});

defineExpose({ loadBookmarks });
</script>

<style scoped>
.bookmarks-panel {
  padding-top: 24px;
}

.bookmark-group {
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 8px;
  overflow: hidden;
}

.group-header {
  display: flex;
  align-items: center;
  padding: 12px 16px;
  background: rgba(255, 255, 255, 0.04);
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}

.conversation-title {
  font-weight: 500;
  font-size: 0.95rem;
  color: rgba(255, 255, 255, 0.9);
}

.bookmark-list {
  background: transparent;
}

.bookmark-item {
  cursor: pointer;
  transition: background-color 0.2s;
}

.bookmark-item:hover {
  background: rgba(187, 134, 252, 0.1);
}

.bookmark-label {
  font-size: 0.9rem;
}

.bookmark-edit-field {
  width: 100%;
}

.bookmark-edit-input :deep(.v-field__input) {
  padding-top: 0;
  padding-bottom: 0;
  min-height: 32px;
  font-size: 0.85rem;
}

.bookmark-date {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.5);
}

.navigate-icon {
  opacity: 0.5;
  transition: opacity 0.2s;
}

.bookmark-item:hover .navigate-icon {
  opacity: 1;
}

.bookmark-action-btn {
  opacity: 0.5;
  transition: opacity 0.2s;
}

.bookmark-item:hover .bookmark-action-btn {
  opacity: 1;
}
</style>

