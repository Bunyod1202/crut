<template>
  <div class="min-h-screen bg-gray-50">
    <!-- Header -->
    <header class="bg-white shadow-sm border-b sticky top-0 z-40">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between items-center py-6">
          <div class="flex items-center">
            <div class="flex-shrink-0">
              <h1 class="text-2xl font-bold text-gray-900 flex items-center">
                <svg
                  class="w-8 h-8 mr-3 text-indigo-600"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M19 20H5a2 2 0 01-2-2V6a2 2 0 012-2h10a2 2 0 012 2v1m2 13a2 2 0 01-2-2V7m2 13a2 2 0 002-2V9a2 2 0 00-2-2h-2m-4-3H9M7 16h6M7 8h6v4H7V8z"
                  ></path>
                </svg>
                Posts Manager
              </h1>
            </div>
          </div>
          <div class="flex items-center space-x-4">
            <div class="text-sm text-gray-500">
              {{ filteredPosts.length }} {{ filteredPosts.length === 1 ? 'post' : 'posts' }}
            </div>
            <button
              @click="openCreateModal"
              class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-lg shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition-all duration-200 transform hover:scale-105"
            >
              <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M12 4v16m8-8H4"
                ></path>
              </svg>
              New Post
            </button>
          </div>
        </div>
      </div>
    </header>

    <!-- Search and Filter Bar -->
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6">
      <div class="bg-white rounded-lg shadow-sm p-4 mb-6">
        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
          <div class="relative">
            <input
              v-model="searchQuery"
              type="text"
              placeholder="Search posts..."
              class="w-full pl-10 pr-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-colors"
            />
            <svg
              class="absolute left-3 top-2.5 h-5 w-5 text-gray-400"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
              ></path>
            </svg>
          </div>

          <select
            v-model="selectedUserId"
            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-colors"
          >
            <option value="">All Users</option>
            <option v-for="userId in uniqueUserIds" :key="userId" :value="userId">
              User {{ userId }}
            </option>
          </select>

          <select
            v-model="sortBy"
            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-colors"
          >
            <option value="id">Sort by ID</option>
            <option value="title">Sort by Title</option>
            <option value="userId">Sort by User</option>
          </select>
        </div>
      </div>
    </div>

    <!-- Main Content -->
    <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pb-12">
      <!-- Loading State -->
      <div v-if="loading" class="flex justify-center items-center py-20">
        <div class="relative">
          <div class="animate-spin rounded-full h-16 w-16 border-b-2 border-indigo-600"></div>
          <div class="absolute inset-0 flex items-center justify-center">
            <div class="animate-pulse h-8 w-8 bg-indigo-200 rounded-full"></div>
          </div>
        </div>
      </div>

      <!-- Error State -->
      <div v-else-if="error" class="bg-red-50 border border-red-200 rounded-lg p-6 mb-6">
        <div class="flex items-center">
          <svg class="h-6 w-6 text-red-400 mr-3" fill="currentColor" viewBox="0 0 20 20">
            <path
              fill-rule="evenodd"
              d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z"
              clip-rule="evenodd"
            ></path>
          </svg>
          <div>
            <h3 class="text-lg font-medium text-red-800">Error loading posts</h3>
            <p class="mt-1 text-sm text-red-700">{{ error }}</p>
            <button
              @click="fetchPosts"
              class="mt-3 inline-flex items-center px-3 py-2 border border-transparent text-sm leading-4 font-medium rounded-md text-red-700 bg-red-100 hover:bg-red-200 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500 transition-colors"
            >
              Try Again
            </button>
          </div>
        </div>
      </div>

      <!-- Posts Grid -->
      <div v-else>
        <div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
          <div
            v-for="post in paginatedPosts"
            :key="post.id"
            class="bg-white rounded-lg shadow-sm hover:shadow-md transition-all duration-200 transform hover:-translate-y-1 border border-gray-200"
          >
            <div class="p-6">
              <div class="flex items-center justify-between mb-4">
                <div class="flex items-center space-x-2">
                  <span
                    class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-indigo-100 text-indigo-800"
                  >
                    #{{ post.id }}
                  </span>
                  <span
                    class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-gray-100 text-gray-800"
                  >
                    <svg class="w-3 h-3 mr-1" fill="currentColor" viewBox="0 0 20 20">
                      <path
                        fill-rule="evenodd"
                        d="M10 9a3 3 0 100-6 3 3 0 000 6zm-7 9a7 7 0 1114 0H3z"
                        clip-rule="evenodd"
                      ></path>
                    </svg>
                    User {{ post.userId }}
                  </span>
                </div>
                <div class="flex space-x-1">
                  <button
                    @click="openEditModal(post)"
                    class="p-2 text-indigo-600 hover:text-indigo-900 hover:bg-indigo-50 rounded-lg transition-colors"
                    title="Edit post"
                  >
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"
                      ></path>
                    </svg>
                  </button>
                  <button
                    @click="confirmDelete(post)"
                    class="p-2 text-red-600 hover:text-red-900 hover:bg-red-50 rounded-lg transition-colors"
                    title="Delete post"
                  >
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
                      ></path>
                    </svg>
                  </button>
                </div>
              </div>

              <h3 class="text-lg font-semibold text-gray-900 mb-3 line-clamp-2 leading-tight">
                {{ post.title }}
              </h3>

              <p class="text-gray-600 text-sm line-clamp-3 leading-relaxed">
                {{ post.body }}
              </p>

              <div class="mt-4 pt-4 border-t border-gray-100">
                <div class="flex items-center justify-between text-xs text-gray-500">
                  <span>{{ formatDate(post.id) }}</span>
                  <span>{{ post.body.length }} characters</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Pagination -->
        <div v-if="totalPages > 1" class="mt-8 flex justify-center">
          <nav class="relative z-0 inline-flex rounded-md shadow-sm -space-x-px">
            <button
              @click="currentPage = Math.max(1, currentPage - 1)"
              :disabled="currentPage === 1"
              class="relative inline-flex items-center px-2 py-2 rounded-l-md border border-gray-300 bg-white text-sm font-medium text-gray-500 hover:bg-gray-50 disabled:opacity-50 disabled:cursor-not-allowed"
            >
              <svg class="h-5 w-5" fill="currentColor" viewBox="0 0 20 20">
                <path
                  fill-rule="evenodd"
                  d="M12.707 5.293a1 1 0 010 1.414L9.414 10l3.293 3.293a1 1 0 01-1.414 1.414l-4-4a1 1 0 010-1.414l4-4a1 1 0 011.414 0z"
                  clip-rule="evenodd"
                ></path>
              </svg>
            </button>

            <button
              v-for="page in visiblePages"
              :key="page"
              @click="currentPage = page"
              :class="[
                'relative inline-flex items-center px-4 py-2 border text-sm font-medium',
                page === currentPage
                  ? 'z-10 bg-indigo-50 border-indigo-500 text-indigo-600'
                  : 'bg-white border-gray-300 text-gray-500 hover:bg-gray-50',
              ]"
            >
              {{ page }}
            </button>

            <button
              @click="currentPage = Math.min(totalPages, currentPage + 1)"
              :disabled="currentPage === totalPages"
              class="relative inline-flex items-center px-2 py-2 rounded-r-md border border-gray-300 bg-white text-sm font-medium text-gray-500 hover:bg-gray-50 disabled:opacity-50 disabled:cursor-not-allowed"
            >
              <svg class="h-5 w-5" fill="currentColor" viewBox="0 0 20 20">
                <path
                  fill-rule="evenodd"
                  d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z"
                  clip-rule="evenodd"
                ></path>
              </svg>
            </button>
          </nav>
        </div>

        <!-- Empty State -->
        <div v-if="filteredPosts.length === 0" class="text-center py-16">
          <svg
            class="mx-auto h-16 w-16 text-gray-400"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="1"
              d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
            ></path>
          </svg>
          <h3 class="mt-4 text-lg font-medium text-gray-900">
            {{ searchQuery || selectedUserId ? 'No posts found' : 'No posts yet' }}
          </h3>
          <p class="mt-2 text-sm text-gray-500">
            {{
              searchQuery || selectedUserId
                ? 'Try adjusting your search or filter criteria.'
                : 'Get started by creating your first post.'
            }}
          </p>
          <button
            v-if="!searchQuery && !selectedUserId"
            @click="openCreateModal"
            class="mt-4 inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          >
            <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M12 4v16m8-8H4"
              ></path>
            </svg>
            Create your first post
          </button>
        </div>
      </div>
    </main>

    <!-- Modal -->
    <div v-if="showModal" class="fixed inset-0 z-50 overflow-y-auto" @click="closeModal">
      <div
        class="flex items-end justify-center min-h-screen pt-4 px-4 pb-20 text-center sm:block sm:p-0"
      >
        <div class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity"></div>

        <div
          class="inline-block align-bottom bg-white rounded-lg text-left overflow-hidden shadow-xl transform transition-all sm:my-8 sm:align-middle sm:max-w-lg sm:w-full"
          @click.stop
        >
          <form @submit.prevent="submitForm">
            <div class="bg-white px-4 pt-5 pb-4 sm:p-6 sm:pb-4">
              <div class="sm:flex sm:items-start">
                <div class="w-full">
                  <div class="flex items-center justify-between mb-4">
                    <h3 class="text-lg leading-6 font-medium text-gray-900">
                      {{ isEditing ? 'Edit Post' : 'Create New Post' }}
                    </h3>
                    <button
                      type="button"
                      @click="closeModal"
                      class="text-gray-400 hover:text-gray-600 transition-colors"
                    >
                      <svg class="h-6 w-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path
                          stroke-linecap="round"
                          stroke-linejoin="round"
                          stroke-width="2"
                          d="M6 18L18 6M6 6l12 12"
                        ></path>
                      </svg>
                    </button>
                  </div>

                  <div class="space-y-4">
                    <div>
                      <label for="title" class="block text-sm font-medium text-gray-700 mb-1">
                        Title <span class="text-red-500">*</span>
                      </label>
                      <input
                        id="title"
                        v-model="form.title"
                        type="text"
                        required
                        maxlength="100"
                        class="block w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm transition-colors"
                        placeholder="Enter post title"
                        :class="{ 'border-red-300': errors.title }"
                      />
                      <div class="flex justify-between mt-1">
                        <p v-if="errors.title" class="text-sm text-red-600">{{ errors.title }}</p>
                        <p class="text-xs text-gray-500 ml-auto">{{ form.title.length }}/100</p>
                      </div>
                    </div>

                    <div>
                      <label for="body" class="block text-sm font-medium text-gray-700 mb-1">
                        Content <span class="text-red-500">*</span>
                      </label>
                      <textarea
                        id="body"
                        v-model="form.body"
                        rows="4"
                        required
                        maxlength="500"
                        class="block w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm transition-colors resize-none"
                        placeholder="Enter post content"
                        :class="{ 'border-red-300': errors.body }"
                      ></textarea>
                      <div class="flex justify-between mt-1">
                        <p v-if="errors.body" class="text-sm text-red-600">{{ errors.body }}</p>
                        <p class="text-xs text-gray-500 ml-auto">{{ form.body.length }}/500</p>
                      </div>
                    </div>

                    <div>
                      <label for="userId" class="block text-sm font-medium text-gray-700 mb-1">
                        User ID <span class="text-red-500">*</span>
                      </label>
                      <input
                        id="userId"
                        v-model.number="form.userId"
                        type="number"
                        required
                        min="1"
                        max="10"
                        class="block w-full border-gray-300 rounded-lg shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm transition-colors"
                        placeholder="Enter user ID (1-10)"
                        :class="{ 'border-red-300': errors.userId }"
                      />
                      <p v-if="errors.userId" class="mt-1 text-sm text-red-600">
                        {{ errors.userId }}
                      </p>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <div class="bg-gray-50 px-4 py-3 sm:px-6 sm:flex sm:flex-row-reverse">
              <button
                type="submit"
                :disabled="submitting || !isFormValid"
                class="w-full inline-flex justify-center rounded-lg border border-transparent shadow-sm px-4 py-2 bg-indigo-600 text-base font-medium text-white hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:ml-3 sm:w-auto sm:text-sm disabled:opacity-50 disabled:cursor-not-allowed transition-all duration-200"
              >
                <svg
                  v-if="submitting"
                  class="animate-spin -ml-1 mr-3 h-5 w-5 text-white"
                  fill="none"
                  viewBox="0 0 24 24"
                >
                  <circle
                    class="opacity-25"
                    cx="12"
                    cy="12"
                    r="10"
                    stroke="currentColor"
                    stroke-width="4"
                  ></circle>
                  <path
                    class="opacity-75"
                    fill="currentColor"
                    d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
                  ></path>
                </svg>
                {{ submitting ? 'Saving...' : isEditing ? 'Update Post' : 'Create Post' }}
              </button>
              <button
                type="button"
                @click="closeModal"
                class="mt-3 w-full inline-flex justify-center rounded-lg border border-gray-300 shadow-sm px-4 py-2 bg-white text-base font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:mt-0 sm:ml-3 sm:w-auto sm:text-sm transition-colors"
              >
                Cancel
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <!-- Delete Confirmation Modal -->
    <div
      v-if="showDeleteModal"
      class="fixed inset-0 z-50 overflow-y-auto"
      @click="closeDeleteModal"
    >
      <div
        class="flex items-end justify-center min-h-screen pt-4 px-4 pb-20 text-center sm:block sm:p-0"
      >
        <div class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity"></div>

        <div
          class="inline-block align-bottom bg-white rounded-lg text-left overflow-hidden shadow-xl transform transition-all sm:my-8 sm:align-middle sm:max-w-lg sm:w-full"
          @click.stop
        >
          <div class="bg-white px-4 pt-5 pb-4 sm:p-6 sm:pb-4">
            <div class="sm:flex sm:items-start">
              <div
                class="mx-auto flex-shrink-0 flex items-center justify-center h-12 w-12 rounded-full bg-red-100 sm:mx-0 sm:h-10 sm:w-10"
              >
                <svg
                  class="h-6 w-6 text-red-600"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-2.5L13.732 4c-.77-.833-1.964-.833-2.732 0L3.732 16.5c-.77.833.192 2.5 1.732 2.5z"
                  ></path>
                </svg>
              </div>
              <div class="mt-3 text-center sm:mt-0 sm:ml-4 sm:text-left">
                <h3 class="text-lg leading-6 font-medium text-gray-900">Delete Post</h3>
                <div class="mt-2">
                  <p class="text-sm text-gray-500">
                    Are you sure you want to delete this post? This action cannot be undone.
                  </p>
                  <div v-if="postToDelete" class="mt-3 p-3 bg-gray-50 rounded-lg">
                    <p class="text-sm font-medium text-gray-900">{{ postToDelete.title }}</p>
                    <p class="text-xs text-gray-600 mt-1 line-clamp-2">{{ postToDelete.body }}</p>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="bg-gray-50 px-4 py-3 sm:px-6 sm:flex sm:flex-row-reverse">
            <button
              @click="deletePost"
              :disabled="deleting"
              class="w-full inline-flex justify-center rounded-lg border border-transparent shadow-sm px-4 py-2 bg-red-600 text-base font-medium text-white hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500 sm:ml-3 sm:w-auto sm:text-sm disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
            >
              <svg
                v-if="deleting"
                class="animate-spin -ml-1 mr-3 h-5 w-5 text-white"
                fill="none"
                viewBox="0 0 24 24"
              >
                <circle
                  class="opacity-25"
                  cx="12"
                  cy="12"
                  r="10"
                  stroke="currentColor"
                  stroke-width="4"
                ></circle>
                <path
                  class="opacity-75"
                  fill="currentColor"
                  d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
                ></path>
              </svg>
              {{ deleting ? 'Deleting...' : 'Delete' }}
            </button>
            <button
              @click="closeDeleteModal"
              class="mt-3 w-full inline-flex justify-center rounded-lg border border-gray-300 shadow-sm px-4 py-2 bg-white text-base font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:mt-0 sm:ml-3 sm:w-auto sm:text-sm transition-colors"
            >
              Cancel
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Toast Notifications -->
    <div
      v-if="toast.show"
      class="fixed bottom-4 right-4 z-50 transform transition-all duration-300"
    >
      <div
        :class="[
          'rounded-lg p-4 shadow-lg max-w-sm',
          toast.type === 'success'
            ? 'bg-green-50 border border-green-200'
            : 'bg-red-50 border border-red-200',
        ]"
      >
        <div class="flex items-center">
          <div class="flex-shrink-0">
            <svg
              v-if="toast.type === 'success'"
              class="h-5 w-5 text-green-400"
              fill="currentColor"
              viewBox="0 0 20 20"
            >
              <path
                fill-rule="evenodd"
                d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z"
                clip-rule="evenodd"
              ></path>
            </svg>
            <svg v-else class="h-5 w-5 text-red-400" fill="currentColor" viewBox="0 0 20 20">
              <path
                fill-rule="evenodd"
                d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z"
                clip-rule="evenodd"
              ></path>
            </svg>
          </div>
          <div class="ml-3">
            <p
              :class="[
                'text-sm font-medium',
                toast.type === 'success' ? 'text-green-800' : 'text-red-800',
              ]"
            >
              {{ toast.message }}
            </p>
          </div>
          <div class="ml-auto pl-3">
            <button
              @click="toast.show = false"
              :class="[
                'inline-flex rounded-md p-1.5 focus:outline-none focus:ring-2 focus:ring-offset-2',
                toast.type === 'success'
                  ? 'text-green-500 hover:bg-green-100 focus:ring-green-600'
                  : 'text-red-500 hover:bg-red-100 focus:ring-red-600',
              ]"
            >
              <svg class="h-4 w-4" fill="currentColor" viewBox="0 0 20 20">
                <path
                  fill-rule="evenodd"
                  d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
                  clip-rule="evenodd"
                ></path>
              </svg>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, reactive, watch } from 'vue'

// Reactive state
const posts = ref([])
const loading = ref(true)
const error = ref(null)
const showModal = ref(false)
const showDeleteModal = ref(false)
const isEditing = ref(false)
const submitting = ref(false)
const deleting = ref(false)
const searchQuery = ref('')
const selectedUserId = ref('')
const sortBy = ref('id')
const currentPage = ref(1)
const postsPerPage = ref(9)
const postToDelete = ref(null)

// Form state
const form = reactive({
  id: null,
  title: '',
  body: '',
  userId: 1,
})

// Form errors
const errors = reactive({
  title: '',
  body: '',
  userId: '',
})

// Toast notification
const toast = reactive({
  show: false,
  message: '',
  type: 'success',
})

// Computed properties
const uniqueUserIds = computed(() => {
  return [...new Set(posts.value.map((post) => post.userId))].sort((a, b) => a - b)
})

const filteredPosts = computed(() => {
  let filtered = [...posts.value]

  // Filter by search query
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    filtered = filtered.filter(
      (post) => post.title.toLowerCase().includes(query) || post.body.toLowerCase().includes(query),
    )
  }

  // Filter by user ID
  if (selectedUserId.value) {
    filtered = filtered.filter((post) => post.userId === parseInt(selectedUserId.value))
  }

  // Sort posts
  filtered.sort((a, b) => {
    switch (sortBy.value) {
      case 'title':
        return a.title.localeCompare(b.title)
      case 'userId':
        return a.userId - b.userId
      default:
        return a.id - b.id
    }
  })

  return filtered
})

const totalPages = computed(() => {
  return Math.ceil(filteredPosts.value.length / postsPerPage.value)
})

const paginatedPosts = computed(() => {
  const start = (currentPage.value - 1) * postsPerPage.value
  const end = start + postsPerPage.value
  return filteredPosts.value.slice(start, end)
})

const visiblePages = computed(() => {
  const pages = []
  const total = totalPages.value
  const current = currentPage.value

  if (total <= 7) {
    for (let i = 1; i <= total; i++) {
      pages.push(i)
    }
  } else {
    if (current <= 4) {
      for (let i = 1; i <= 5; i++) {
        pages.push(i)
      }
      pages.push('...', total)
    } else if (current >= total - 3) {
      pages.push(1, '...')
      for (let i = total - 4; i <= total; i++) {
        pages.push(i)
      }
    } else {
      pages.push(1, '...')
      for (let i = current - 1; i <= current + 1; i++) {
        pages.push(i)
      }
      pages.push('...', total)
    }
  }

  return pages.filter((page) => page !== '...' || pages.indexOf(page) === pages.lastIndexOf(page))
})

const isFormValid = computed(() => {
  return (
    form.title.trim() &&
    form.body.trim() &&
    form.userId > 0 &&
    !errors.title &&
    !errors.body &&
    !errors.userId
  )
})

// Watchers
watch(
  () => [searchQuery.value, selectedUserId.value],
  () => {
    currentPage.value = 1
  },
)

watch(
  () => form.title,
  (newTitle) => {
    if (newTitle.length > 100) {
      errors.title = 'Title must be 100 characters or less'
    } else if (!newTitle.trim()) {
      errors.title = 'Title is required'
    } else {
      errors.title = ''
    }
  },
)

watch(
  () => form.body,
  (newBody) => {
    if (newBody.length > 500) {
      errors.body = 'Content must be 500 characters or less'
    } else if (!newBody.trim()) {
      errors.body = 'Content is required'
    } else {
      errors.body = ''
    }
  },
)

watch(
  () => form.userId,
  (newUserId) => {
    if (!newUserId || newUserId < 1) {
      errors.userId = 'User ID must be a positive number'
    } else if (newUserId > 10) {
      errors.userId = 'User ID must be between 1 and 10'
    } else {
      errors.userId = ''
    }
  },
)

// Methods
const fetchPosts = async () => {
  try {
    loading.value = true
    error.value = null
    const response = await fetch('https://jsonplaceholder.typicode.com/posts')
    if (!response.ok) throw new Error('Failed to fetch posts')
    posts.value = await response.json()
  } catch (err) {
    error.value = err.message
    showToast('Failed to load posts', 'error')
  } finally {
    loading.value = false
  }
}

const openCreateModal = () => {
  isEditing.value = false
  Object.assign(form, {
    id: null,
    title: '',
    body: '',
    userId: 1,
  })
  Object.assign(errors, {
    title: '',
    body: '',
    userId: '',
  })
  showModal.value = true
}

const openEditModal = (post) => {
  isEditing.value = true
  Object.assign(form, { ...post })
  Object.assign(errors, {
    title: '',
    body: '',
    userId: '',
  })
  showModal.value = true
}

const closeModal = () => {
  showModal.value = false
  Object.assign(form, {
    id: null,
    title: '',
    body: '',
    userId: 1,
  })
  Object.assign(errors, {
    title: '',
    body: '',
    userId: '',
  })
}

const submitForm = async () => {
  if (!isFormValid.value) return

  try {
    submitting.value = true

    if (isEditing.value) {
      await updatePost()
    } else {
      await createPost()
    }

    closeModal()
  } catch (err) {
    console.error(err)
    showToast('Operation failed', 'error')
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
  newPost.id = Date.now() // Use unique ID since JSONPlaceholder returns 101
  posts.value.unshift(newPost)
  showToast('Post created successfully!', 'success')
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
  showToast('Post updated successfully!', 'success')
}

const confirmDelete = (post) => {
  postToDelete.value = post
  showDeleteModal.value = true
}

const closeDeleteModal = () => {
  showDeleteModal.value = false
  postToDelete.value = null
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
    closeDeleteModal()
    showToast('Post deleted successfully!', 'success')
  } catch (err) {
    console.error(err)
    showToast('Failed to delete post', 'error')
  } finally {
    deleting.value = false
  }
}

const showToast = (message, type = 'success') => {
  toast.show = true
  toast.message = message
  toast.type = type

  setTimeout(() => {
    toast.show = false
  }, 4000)
}

const formatDate = (id) => {
  // Mock date formatting based on ID
  const date = new Date(2024, 0, (id % 30) + 1)
  return date.toLocaleDateString('en-US', {
    month: 'short',
    day: 'numeric',
    year: 'numeric',
  })
}

// Lifecycle
onMounted(() => {
  fetchPosts()
})
</script>

<style scoped>
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.line-clamp-3 {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

input,
textarea,
select {
  border: 1px solid #d1d5db;
  padding: 0.5rem 0.75rem;
  transition: all 0.2s ease;
}

input:focus,
textarea:focus,
select:focus {
  outline: none;
  border-color: #6366f1;
  box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.1);
}

.transform {
  transition: transform 0.2s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fadeIn {
  animation: fadeIn 0.3s ease-out;
}
</style>
