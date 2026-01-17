class PhotoChatApp {
  constructor() {
    this.photos = JSON.parse(localStorage.getItem('photos')) || [];
    this.messages = JSON.parse(localStorage.getItem('messages')) || [];
    this.init();
  }

  init() {
    this.setupListeners();
    this.displayMessages();
    this.displayPhotos();
  }

  setupListeners() {
    document.getElementById('photoInput').addEventListener('change', (e) => this.handlePhotoSelect(e));
    document.getElementById('messageInput').addEventListener('keypress', (e) => {
      if (e.key === 'Enter') this.sendMessage();
    });
  }

  handlePhotoSelect(e) {
    const file = e.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (event) => {
      document.photoData = event.target.result;
      console.log('Photo selected');
    };
    reader.readAsDataURL(file);
  }

  uploadPhoto() {
    if (!document.photoData) {
      alert('Please select a photo first');
      return;
    }

    const caption = document.getElementById('caption').value.trim();
    const photo = {
      id: Date.now(),
      image: document.photoData,
      caption: caption,
      timestamp: new Date().toLocaleString()
    };

    this.photos.unshift(photo);
    this.savePhotos();
    this.displayPhotos();
    document.getElementById('caption').value = '';
    document.getElementById('photoInput').value = '';
    document.photoData = null;
    alert('Photo uploaded!');
  }

  sendMessage() {
    const input = document.getElementById('messageInput');
    const text = input.value.trim();

    if (!text) return;

    const message = {
      id: Date.now(),
      text: text,
      sender: 'you',
      timestamp: new Date().toLocaleTimeString()
    };

    this.messages.push(message);
    this.saveMessages();
    this.displayMessages();
    input.value = '';

    // Auto-reply
    setTimeout(() => {
      const reply = {
        id: Date.now() + 1,
        text: '👍 Thanks for chatting!',
        sender: 'other',
        timestamp: new Date().toLocaleTimeString()
      };
      this.messages.push(reply);
      this.saveMessages();
      this.displayMessages();
    }, 1000);
  }

  displayMessages() {
    const chatMessages = document.getElementById('chatMessages');
    
    if (this.messages.length === 0) {
      chatMessages.innerHTML = '<div class="empty-message">No messages yet. Start chatting!</div>';
      return;
    }

    chatMessages.innerHTML = this.messages.map(msg => `
      <div class="chat-message ${msg.sender}">
        <strong>${msg.sender === 'you' ? 'You' : 'Other'}</strong><br>
        ${this.escape(msg.text)}<br>
        <small>${msg.timestamp}</small>
      </div>
    `).join('');

    chatMessages.scrollTop = chatMessages.scrollHeight;
  }

  displayPhotos() {
    const feed = document.getElementById('photoFeed');

    if (this.photos.length === 0) {
      feed.innerHTML = '<div class="empty-message">No photos yet. Share one!</div>';
      return;
    }

    feed.innerHTML = this.photos.map(photo => `
      <div class="photo-post">
        <img src="${photo.image}" alt="photo" class="photo-image">
        ${photo.caption ? `<div class="photo-caption">${this.escape(photo.caption)}</div>` : ''}
        <div class="photo-meta">${photo.timestamp}</div>
      </div>
    `).join('');
  }

  savePhotos() {
    localStorage.setItem('photos', JSON.stringify(this.photos));
  }

  saveMessages() {
    localStorage.setItem('messages', JSON.stringify(this.messages));
  }

  escape(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
  }
}

const app = new PhotoChatApp();
