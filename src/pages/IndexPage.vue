<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-toolbar-title class="flex items-center">
          <q-icon name="article" size="md" class="q-mr-sm" />
          Posts Manager
          <q-chip
            v-if="!loading"
            :label="`${filteredPosts.length} posts`"
            color="white"
            text-color="primary"
            size="sm"
            class="q-ml-md"
          />
        </q-toolbar-title>
        <q-space />
        <q-btn
          @click="openCreateDialog"
          color="white"
          text-color="primary"
          label="New Post"
          icon="add"
          unelevated
          no-caps
          class="q-mr-sm"
        />
        <q-btn @click="fetchPosts" icon="refresh" flat round :loading="loading">
          <q-tooltip>Refresh posts</q-tooltip>
        </q-btn>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="q-pa-md">
        <div v-if="loading" class="flex flex-center q-pa-xl">
          <div class="text-center">
            <q-spinner-dots color="primary" size="50px" />
            <div class="text-h6 q-mt-md text-grey-6">Loading posts...</div>
          </div>
        </div>

        <q-banner v-else-if="error" class="bg-negative text-white q-mb-md" rounded>
          <template v-slot:avatar>
            <q-icon name="error" />
          </template>
          <div class="text-body1">{{ error }}</div>
          <template v-slot:action>
            <q-btn flat label="Retry" @click="fetchPosts" />
          </template>
        </q-banner>
        <div v-else>
          <q-intersection
            v-for="post in paginatedPosts"
            :key="post.id"
            once
            class="intersection-item"
          >
            <q-card class="post-card q-mb-md cursor-pointer" bordered @click="viewPost(post)">
              <q-card-section class="q-pb-none">
                <div class="row items-center justify-between">
                  <div class="row items-center q-gutter-xs">
                    <q-chip
                      :label="`#${post.id}`"
                      color="primary"
                      text-color="white"
                      size="sm"
                      dense
                    />
                    <q-chip
                      :label="`User ${post.userId}`"
                      color="grey-3"
                      text-color="grey-8"
                      size="sm"
                      dense
                      icon="person"
                    />
                  </div>

                  <div class="row q-gutter-xs">
                    <q-btn
                      @click.stop="openEditDialog(post)"
                      icon="edit"
                      color="primary"
                      flat
                      round
                      size="sm"
                    >
                      <q-tooltip>Edit post</q-tooltip>
                    </q-btn>

                    <q-btn
                      @click.stop="confirmDelete(post)"
                      icon="delete"
                      color="negative"
                      flat
                      round
                      size="sm"
                    >
                      <q-tooltip>Delete post</q-tooltip>
                    </q-btn>
                  </div>
                </div>
              </q-card-section>

              <q-card-section>
                <div class="text-h6 q-mb-sm text-weight-medium ellipsis-2-lines">
                  {{ post.title }}
                </div>
                <div class="text-body2 text-grey-7 ellipsis-3-lines">
                  {{ post.body }}
                </div>
              </q-card-section>

              <q-card-section class="q-pt-none">
                <div class="row items-center justify-between">
                  <q-chip
                    :label="`${post.body.length} chars`"
                    color="grey-2"
                    text-color="grey-7"
                    size="sm"
                    dense
                    icon="text_fields"
                  />
                  <q-chip
                    :label="formatDate(post.id)"
                    color="grey-2"
                    text-color="grey-7"
                    size="sm"
                    dense
                    icon="schedule"
                  />
                </div>
              </q-card-section>

              <q-card-actions align="right" class="q-pt-none">
                <q-btn
                  flat
                  color="primary"
                  label="View Details"
                  size="sm"
                  @click.stop="viewPost(post)"
                />
              </q-card-actions>
            </q-card>
          </q-intersection>
          <div v-if="totalPages > 1" class="flex flex-center q-mt-lg">
            <q-pagination
              v-model="currentPage"
              :max="totalPages"
              :max-pages="6"
              direction-links
              boundary-links
              color="primary"
              active-design="unelevated"
              active-color="primary"
              active-text-color="white"
            />
          </div>
          <q-card v-if="filteredPosts.length === 0" flat class="text-center q-pa-xl">
            <q-card-section>
              <q-icon name="inbox" size="80px" color="grey-4" />
              <q-btn
                @click="openCreateDialog"
                color="primary"
                label="Create Your First Post"
                icon="add"
                class="q-mt-md"
                unelevated
              />
            </q-card-section>
          </q-card>
        </div>
      </q-page>
    </q-page-container>
    <q-dialog v-model="showDialog" persistent>
      <q-card style="min-width: 500px; max-width: 600px">
        <q-card-section class="row items-center q-pb-none">
          <div class="text-h6">
            <q-icon :name="isEditing ? 'edit' : 'add'" class="q-mr-sm" />
            {{ isEditing ? 'Edit Post' : 'Create New Post' }}
          </div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>
        <q-form @submit="submitForm" class="q-gutter-md">
          <q-card-section>
            <q-input
              v-model="form.title"
              label="Title"
              outlined
              :rules="[(val) => !!val || 'Title is required']"
              maxlength="100"
              counter
              autofocus
            >
              <template v-slot:prepend>
                <q-icon name="title" />
              </template>
            </q-input>
            <q-input
              v-model="form.body"
              label="Content"
              type="textarea"
              outlined
              rows="5"
              :rules="[(val) => !!val || 'Content is required']"
              maxlength="500"
              counter
            >
              <template v-slot:prepend>
                <q-icon name="description" />
              </template>
            </q-input>
            <q-input
              v-model.number="form.userId"
              label="User ID"
              type="number"
              outlined
              :rules="[
                (val) => !!val || 'User ID is required',
                (val) => val > 0 || 'User ID must be positive',
                (val) => val <= 10 || 'User ID must be between 1 and 10',
              ]"
              min="1"
              max="10"
            >
              <template v-slot:prepend>
                <q-icon name="person" />
              </template>
            </q-input>
          </q-card-section>
          <q-card-actions align="right" class="q-pa-md">
            <q-btn flat label="Cancel" v-close-popup />
            <q-btn
              type="submit"
              :label="isEditing ? 'Update Post' : 'Create Post'"
              color="primary"
              :loading="submitting"
              unelevated
            >
              <template v-slot:loading>
                <q-spinner-hourglass class="on-left" />
                {{ isEditing ? 'Updating...' : 'Creating...' }}
              </template>
            </q-btn>
          </q-card-actions>
        </q-form>
      </q-card>
    </q-dialog>
    <q-dialog v-model="showViewDialog">
      <q-card style="min-width: 500px; max-width: 700px">
        <q-card-section v-if="selectedPost">
          <div class="row items-center justify-between q-mb-md">
            <div class="text-h6">Post Details</div>
            <q-btn icon="close" flat round dense v-close-popup />
          </div>

          <div class="row q-gutter-sm q-mb-md">
            <q-chip
              :label="`ID: ${selectedPost.id}`"
              color="primary"
              text-color="white"
              icon="tag"
            />
            <q-chip
              :label="`User: ${selectedPost.userId}`"
              color="secondary"
              text-color="white"
              icon="person"
            />
            <q-chip
              :label="`${selectedPost.body.length} characters`"
              color="grey-3"
              text-color="grey-8"
              icon="text_fields"
            />
          </div>
          <div class="text-h5 q-mb-md text-weight-medium">
            {{ selectedPost.title }}
          </div>
          <div class="text-body1 text-grey-8 line-height-md">
            {{ selectedPost.body }}
          </div>
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="Close" v-close-popup />
          <q-btn
            @click="openEditDialog(selectedPost)"
            color="primary"
            label="Edit"
            icon="edit"
            unelevated
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
    <q-dialog v-model="showDeleteDialog" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar icon="delete" color="negative" text-color="white" />
          <span class="q-ml-sm text-h6">Confirm Deletion</span>
        </q-card-section>

        <q-card-section v-if="postToDelete">
          <div class="text-body1 q-mb-sm">
            Are you sure you want to delete this post? This action cannot be undone.
          </div>
          <q-card flat bordered class="bg-grey-1">
            <q-card-section>
              <div class="text-weight-medium">{{ postToDelete.title }}</div>
              <div class="text-body2 text-grey-6 q-mt-xs ellipsis-2-lines">
                {{ postToDelete.body }}
              </div>
            </q-card-section>
          </q-card>
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="Cancel" v-close-popup />
          <q-btn label="Delete" color="negative" @click="deletePost" :loading="deleting" unelevated>
            <template v-slot:loading>
              <q-spinner-hourglass class="on-left" />
              Deleting...
            </template>
          </q-btn>
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-layout>
</template>

<script setup>
import { ref, computed, onMounted, reactive } from 'vue'

const posts = ref([])
const loading = ref(true)
const error = ref(null)
const showDialog = ref(false)
const showViewDialog = ref(false)
const showDeleteDialog = ref(false)
const isEditing = ref(false)
const submitting = ref(false)
const deleting = ref(false)
const currentPage = ref(1)
const postToDelete = ref(null)
const selectedPost = ref(null)

const form = reactive({
  id: null,
  title: '',
  body: '',
  userId: 1,
})

const filteredPosts = computed(() => {
  let filtered = [...posts.value]
  return filtered
})

const totalPages = computed(() => {
  return Math.ceil(filteredPosts.value.length / 6)
})

const paginatedPosts = computed(() => {
  const start = (currentPage.value - 1) * 6
  const end = start + 6
  return filteredPosts.value.slice(start, end)
})

const fetchPosts = async () => {
  try {
    loading.value = true
    const response = await fetch('https://jsonplaceholder.typicode.com/posts')
    if (!response.ok) throw new Error('Failed to fetch posts')
    posts.value = await response.json()
    console.log(`Loaded ${posts.value.length} posts successfully`)
  } catch (err) {
    console.error('Failed to load posts', err)
  } finally {
    loading.value = false
  }
}

const openCreateDialog = () => {
  isEditing.value = false
  Object.assign(form, {
    id: null,
    title: '',
    body: '',
    userId: 1,
  })
  showDialog.value = true
}

const openEditDialog = (post) => {
  isEditing.value = true
  Object.assign(form, { ...post })
  showViewDialog.value = false
  showDialog.value = true
}

const viewPost = (post) => {
  selectedPost.value = post
  showViewDialog.value = true
}

const submitForm = async () => {
  try {
    submitting.value = true

    if (isEditing.value) {
      await updatePost()
    } else {
      await createPost()
    }

    showDialog.value = false
  } catch (err) {
    console.error('Failed to submit form', err)
  } finally {
    submitting.value = false
  }
}

const createPost = async () => {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
    method: 'POST',
    body: JSON.stringify({
      title: form.title,
      body: form.body,
      userId: form.userId,
    }),
    headers: {
      'Content-type': 'application/json; charset=UTF-8',
    },
  })

  if (!response.ok) throw new Error('Failed to create post')

  const newPost = await response.json()
  newPost.id = Date.now()
  posts.value.unshift(newPost)

  console.log('Post created successfully!', newPost)
}

const updatePost = async () => {
  const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${form.id}`, {
    method: 'PUT',
    body: JSON.stringify({
      id: form.id,
      title: form.title,
      body: form.body,
      userId: form.userId,
    }),
    headers: {
      'Content-type': 'application/json; charset=UTF-8',
    },
  })

  if (!response.ok) throw new Error('Failed to update post')

  const updatedPost = await response.json()
  const index = posts.value.findIndex((p) => p.id === form.id)
  if (index !== -1) {
    posts.value.splice(index, 1, updatedPost)
  }
  console.log('Post updated successfully!', updatedPost)
}

const confirmDelete = (post) => {
  postToDelete.value = post
  showDeleteDialog.value = true
}

const deletePost = async () => {
  try {
    deleting.value = true
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/posts/${postToDelete.value.id}`,
      {
        method: 'DELETE',
      },
    )

    if (!response.ok) throw new Error('Failed to delete post')

    posts.value = posts.value.filter((post) => post.id !== postToDelete.value.id)
    showDeleteDialog.value = false

    console.log('Post deleted successfully!', postToDelete.value)
  } catch (err) {
    console.error('Failed to delete post', err)
  } finally {
    deleting.value = false
  }
}

const formatDate = (id) => {
  const date = new Date(2024, 0, (id % 30) + 1)
  return date.toLocaleDateString('en-US', {
    month: 'short',
    day: 'numeric',
  })
}

onMounted(() => {
  fetchPosts()
})
</script>

<style scoped>
.post-card {
  transition: all 0.3s ease;
  border-radius: 12px;
}

.post-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.ellipsis-2-lines {
  display: -webkit-box;
  overflow: hidden;
}

.ellipsis-3-lines {
  display: -webkit-box;
  overflow: hidden;
}

.intersection-item {
  transition: opacity 0.6s ease;
}

.line-height-md {
  line-height: 1.6;
}

.q-card {
  border-radius: 12px;
}

.q-btn {
  border-radius: 8px;
}
</style>
