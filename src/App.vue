<template>
  <div class="user-columns">
    <div v-for="user in users" :key="user.id" class="user-column">
      <h3 class="user-name">{{ user.name }} ({{ user.posts.length }} Posts)</h3>

      <draggable :list="user.posts" group="users" :itemKey="post => post.id" @change="handleDrag(user)">
        <template v-slot:item="{ element, index }">
          <div class="user-item" :key="element.id">
            {{ element.title }}
          <div class="user-control">
        <button @click="openModal(user.id, element.id)">Edit </button>
            <button @click="deletePost(user.id, element.id)">Delete</button>

          </div>
          </div>
        </template>
      </draggable>
      <form @submit.prevent="addPost(user)">
        <input class="user-input" v-model="user.newPostTitle" placeholder="New Post Title" />
        <button class="user-button" type="submit">Add Post</button>
      </form>
    </div>
    <div v-if="isModalVisible" class="modal">
      <h3>Edit Post</h3>
      <input v-model="editPosts.title" placeholder="Title" />
      <textarea v-model="editPosts.descr" placeholder="Description"></textarea>
      <button @click="editPost(currentlyEditingUser.id, currentlyEditingPost.id)">Save Changes</button>

      <button @click="closeModal">Cancel</button>
    </div>
  </div>

</template>

<script setup>
import { ref ,watchEffect} from 'vue';
import draggable from 'vuedraggable';



const users = ref([]);
const editPosts = ref({
  title: '',
  descr: '',
});
const currentlyEditingUser = ref(null);
const currentlyEditingPost = ref(null);
const isModalVisible = ref(false);

const getUsers = async () => {
  const response = await fetch('https://jsonplaceholder.typicode.com/users');
  const data = await response.json();
  users.value = data.map(user => ({
    ...user,
    posts: [],
    newPostTitle: '',
  }));
};

const handleDrag = user => event => {
  const draggedPost = event.item;
  const oldIndex = event.oldIndex;
  const newIndex = event.newIndex;

  user.posts.splice(oldIndex, 1);

  if (newIndex !== null) {
    const targetUser = users.value[newIndex];
    targetUser.posts.splice(newIndex, 0, draggedPost);
  } else {
    user.posts.push(draggedPost);
  }
};

const addPost = user => {
  if (user.newPostTitle) {
    const newPost = {
      id: Date.now(),
      title: user.newPostTitle,
      descr: '',
    };
    user.posts.push(newPost);
    user.newPostTitle = '';
  }
};

const openModal = (userId, postId) => {
  currentlyEditingUser.value = userId;
  currentlyEditingPost.value = postId;
  isModalVisible.value = true;
  const post = users.value.find(user => user.id === userId)?.posts.find(post => post.id === postId);
  editPosts.title = post ? post.title : '';
  editPosts.descr = post ? post.descr : '';
};


const closeModal = () => {
  isModalVisible.value = false;
  currentlyEditingUser.value = null;
  currentlyEditingPost.value = null;
};
const editPost = (userId, postId) => {
  const userToUpdate = users.value.find(user => user.id === userId);

  if (userToUpdate) {
    const postToUpdate = userToUpdate.posts.find(post => post.id === postId);

    if (postToUpdate) {
      postToUpdate.title = editPosts.title;
      postToUpdate.descr = editPosts.descr;
      closeModal();
    }
  }
};
const deletePost = (userId, postId) => {
  const userToUpdate = users.value.find(user => user.id === userId);

  if (userToUpdate) {
    const postIndex = userToUpdate.posts.findIndex(post => post.id === postId);

    if (postIndex !== -1) {
      userToUpdate.posts.splice(postIndex, 1);
      closeModal();
    }
  }
};
watchEffect(() => {
  getUsers();
});

</script>

<style>
.user-columns {
  display: flex;
  max-width: 100vw;
  min-height: 100vh;

}

.user-column {
  width: 10%;
  border: 1px solid #ccc;
  display: flex;
  flex-direction: column;
  background-color: #fff;

  gap: 15px;
}

.user-name {
  font-size: 14px;
}

.user-item {
  box-sizing: border-box;
  border: 1px solid #ccc;
  background-color: #fff;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.user-control{
display: flex;
justify-content: space-between;
}

.user-input {
  width: 80%;
}

.modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding: 20px;
  background-color: #fff;
  border: 1px solid #ccc;
  box-shadow: 0px 2px 10px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
}
</style>
