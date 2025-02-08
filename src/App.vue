<template>
  <div class="mail-container">
    <header class="mail-header">
      <h1>MyMail</h1>
      <nav>
        <ul>
          <li v-if="!isLoggedIn"><a href="#" @click="showLogin">Login</a></li>
          <li v-if="isLoggedIn"><a href="#" @click="logout">Logout</a></li>
        </ul>
      </nav>
    </header>
    
    <div class="mail-body">
      <div class="login-form" v-if="!isLoggedIn">
        <h2>Login</h2>
        <input type="email" v-model="email" placeholder="Email" class="input-field" />
        <input type="password" v-model="password" placeholder="Password" class="input-field" />
        <button @click="handleLogin" class="btn">Login</button>
      </div>
      
      <div class="inbox" v-else>
        <h2>Inbox</h2>
        <div class="email-list">
          <div class="email-item" v-for="(email, index) in emails" :key="index" @click="openEmail(email)">
            <p><strong>From:</strong> {{ email.from }}</p>
            <p><strong>Subject:</strong> {{ email.subject }}</p>
            <p class="preview">{{ email.preview }}</p>
          </div>
        </div>
        
        <div v-if="selectedEmail" class="email-content">
          <h2>{{ selectedEmail.subject }}</h2>
          <p><strong>From:</strong> {{ selectedEmail.from }}</p>
          <p>{{ selectedEmail.preview }}</p>
          <button @click="closeEmail" class="btn">Close</button>
        </div>
        
        <div class="send-email">
          <h2>Send Email</h2>
          <input type="text" v-model="newEmail.from" placeholder="From" class="input-field" />
          <input type="text" v-model="newEmail.subject" placeholder="Subject" class="input-field" />
          <textarea v-model="newEmail.preview" placeholder="Message" class="input-field"></textarea>
          <button @click="sendEmail" class="btn">Send</button>
        </div>
        
        <div class="excel-upload">
          <h2>Upload Excel File</h2>
          <input type="file" @change="handleFileUpload" accept=".xlsx, .xls" class="input-file" />
          <button @click="saveDataToAPI" class="btn">Save to API</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as XLSX from "xlsx";

export default {
  data() {
    return {
      email: "",
      password: "",
      isLoggedIn: false,
      emails: [],
      selectedEmail: null,
      newEmail: {
        from: "",
        subject: "",
        preview: ""
      },
      parsedData: [],
      apiUrl: "https://api.mokky.dev/YOUR_PROJECT_ID/emails",
      excelApiUrl: "https://api.mokky.dev/YOUR_PROJECT_ID/excelData"
    };
  },
  methods: {
    async fetchEmails() {
      try {
        const response = await fetch(this.apiUrl);
        this.emails = await response.json();
      } catch (error) {
        console.error("Ошибка загрузки писем:", error);
      }
    },
    async sendEmail() {
      try {
        const response = await fetch(this.apiUrl, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(this.newEmail)
        });
        if (response.ok) {
          this.newEmail = { from: "", subject: "", preview: "" };
          this.fetchEmails();
        }
      } catch (error) {
        console.error("Ошибка отправки письма:", error);
      }
    },
    handleLogin() {
      if (this.email && this.password) {
        this.isLoggedIn = true;
        this.fetchEmails();
      } else {
        alert("Please enter valid credentials.");
      }
    },
    logout() {
      this.isLoggedIn = false;
      this.email = "";
      this.password = "";
    },
    showLogin() {
      this.isLoggedIn = false;
    },
    openEmail(email) {
      this.selectedEmail = email;
    },
    closeEmail() {
      this.selectedEmail = null;
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      const reader = new FileReader();
      
      reader.onload = (e) => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: "array" });
        const sheetName = workbook.SheetNames[0];
        const sheet = workbook.Sheets[sheetName];
        this.parsedData = XLSX.utils.sheet_to_json(sheet);
      };
      
      reader.readAsArrayBuffer(file);
    },
    async saveDataToAPI() {
      try {
        await fetch(this.excelApiUrl, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(this.parsedData)
        });
        alert("Excel data saved successfully!");
      } catch (error) {
        console.error("Ошибка сохранения данных Excel:", error);
      }
    }
  }
};
</script>

<style scoped>
.mail-container {
  font-family: Arial, sans-serif;
  background-color: #f3f3f3;
  min-height: 100vh;
}

.mail-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  background: #0078d4;
  color: white;
}

.mail-body {
  max-width: 600px;
  margin: 20px auto;
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.input-field, .input-file {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.btn {
  width: 100%;
  padding: 10px;
  background: #0078d4;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn:hover {
  background: #005a9e;
}
</style>
