# Svelte + PocketBase Chat App

This project is a simple chat application built with Svelte, TypeScript, Vite, and [PocketBase](https://pocketbase.io/) as the backend.

## Features

- User authentication (login/signup)
- Real-time messaging
- User avatars
- Responsive UI

## PocketBase Setup

1. **Download PocketBase**  
   Download the [PocketBase](https://pocketbase.io/docs/) binary for your OS and place it in the `pb_data/` directory (already included as `pocketbase.exe`).

2. **Start the PocketBase Server**  
   Open a terminal in the `pb_data/` directory and run:
   ```sh
   ./pocketbase serve
   ```
   This will start the PocketBase server at [http://127.0.0.1:8090](http://127.0.0.1:8090).

3. **Access the Admin UI**  
   Visit [http://127.0.0.1:8090/_/](http://127.0.0.1:8090/_/) in your browser to access the PocketBase admin dashboard.

4. **Create Collections**  
   - **users**: For authentication (PocketBase creates this by default).
   - **messages**: For storing chat messages.  
     Fields:
     - `text` (type: text, required)
     - `user` (type: relation, required, points to users collection)
     - `created` (type: autodate, system field)

5. **Configure Rules**  
   Set appropriate create/list/view rules for the `messages` collection to allow authenticated users to send and receive messages.

6. **Run the Frontend**  
   In the `messageApp/` directory, install dependencies and start the dev server:
   ```sh
   npm install
   npm run dev
   ```
   The app will be available at [http://localhost:5173](http://localhost:5173) (or as indicated in your terminal).

## UI Screenshots

Below are example screenshots of the application UI. Place your images in the `Images/` folder.

### Login Screen
![Login](../Practice/Images/Login.png)

### Signup Screen
![Signup](../Practice/Images/Signup.png)

### Chat Interface
![Chat Interface](../Practice/Images/Interface.png)

---

## Recommended IDE Setup

- [VS Code](https://code.visualstudio.com/) + [Svelte Extension](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode)

---

## License

MIT

## Credits
Fireship