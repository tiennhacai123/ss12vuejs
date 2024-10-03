<template>
  <div class="container">
    <div class="header">
      <h1>Quản lý mượn trả sách</h1>

      <select v-model="selectedFilter" @change="filterBooks" class="filter-select">
        <option value="">Tất cả</option>
        <option value="Đã trả">Đã trả</option>
        <option value="Chưa trả">Chưa trả</option>
      </select>

      <button @click="showAddForm" class="add-button">Thêm thông tin</button>
    </div>

    <table class="book-table">
      <thead>
        <tr>
          <th>STT</th>
          <th>Tên sách</th>
          <th>Sinh viên mượn</th>
          <th>Ngày mượn</th>
          <th>Ngày trả</th>
          <th>Trạng thái</th>
          <th>Chức năng</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(book, index) in filteredBooks" :key="index">
          <td>{{ index + 1 }}</td>
          <td>{{ book.title }}</td>
          <td>{{ book.borrower }}</td>
          <td>{{ book.borrowDate }}</td>
          <td>{{ book.returnDate }}</td>
          <td>
            <select v-model="book.status" @change="saveBooksToLocalStorage">
              <option value="Đã trả">Đã trả</option>
              <option value="Chưa trả">Chưa trả</option>
            </select>
          </td>
          <td>
            <button @click="editBook(index)" class="edit-button">Sửa</button>
            <button @click="confirmDeleteBook(index)" class="delete-button">Xóa</button>
          </td>
        </tr>
      </tbody>
    </table>

    <div class="overlay" v-if="isAddFormVisible">
      <form class="form" @submit.prevent="addBook">
        <h4>{{ isEditing ? 'Chỉnh sửa thông tin sách' : 'Thêm mới sách' }}</h4>
        <input v-model="newBook.title" type="text" placeholder="Tên sách" required />
        <input v-model="newBook.borrower" type="text" placeholder="Tên người mượn" required />
        <input v-model="newBook.borrowDate" type="date" required />
        <input v-model="newBook.returnDate" type="date" required />
        <button type="submit" class="submit-button">Thêm mới</button>
        <button @click="closeForm" type="button" class="cancel-button">Hủy</button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const isAddFormVisible = ref(false);
const isEditing = ref(false);
const books = ref([]);
const filteredBooks = ref([]);
const newBook = ref({ title: '', borrower: '', borrowDate: '', returnDate: '', status: 'Chưa trả' });
const editingIndex = ref(null);
const selectedFilter = ref('');

onMounted(() => {
  const savedBooks = localStorage.getItem('books');
  if (savedBooks) books.value = JSON.parse(savedBooks);
  filterBooks();
});

const showAddForm = () => { isAddFormVisible.value = true; };
const closeForm = () => { isAddFormVisible.value = false; resetForm(); };
const addBook = () => {
  if (isEditing.value) {
    books.value.splice(editingIndex.value, 1, { ...newBook.value });
    isEditing.value = false;
  } else {
    books.value.push({ ...newBook.value });
  }
  saveBooksToLocalStorage();
  closeForm();
};
const editBook = (index) => {
  isEditing.value = true;
  newBook.value = { ...books.value[index] };
  editingIndex.value = index;
  isAddFormVisible.value = true;
};
const confirmDeleteBook = (index) => {
  if (confirm("Bạn có chắc chắn muốn xóa sách này?")) {
    deleteBook(index);
    alert("Sách đã được xóa");
  }
};
const deleteBook = (index) => { books.value.splice(index, 1); saveBooksToLocalStorage(); filterBooks(); };
const resetForm = () => { newBook.value = { title: '', borrower: '', borrowDate: '', returnDate: '', status: 'Chưa trả' }; };
const saveBooksToLocalStorage = () => { localStorage.setItem('books', JSON.stringify(books.value)); filterBooks(); };
const filterBooks = () => { filteredBooks.value = selectedFilter.value ? books.value.filter(book => book.status === selectedFilter.value) : books.value; };
</script>

<style scoped>
.container {
  width: 90%;
  margin: 0 auto;
  padding: 20px;
  background-color: #f4f4f9;
  border-radius: 10px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.filter-select {
  padding: 8px;
  border-radius: 5px;
  border: 1px solid #ccc;
}

.add-button {
  background-color: #28a745;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.book-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

.book-table th, .book-table td {
  border: 1px solid #ddd;
  padding: 12px;
  text-align: center;
}

.book-table th {
  background-color: #f2f2f2;
}

.edit-button {
  background-color: #ffc107;
  color: white;
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.delete-button {
  background-color: #dc3545;
  color: white;
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
}

.form {
  background: white;
  padding: 20px;
  border-radius: 10px;
  width: 400px;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.form input {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  width: 100%;
}

.submit-button {
  background-color: #007bff;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.cancel-button {
  background-color: #6c757d;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
</style>
