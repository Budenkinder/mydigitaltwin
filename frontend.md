```markdown
# Frontend Implementation Guide: AI Digital Twin SaaS

**Version:** 1.0
**Date:** June 2, 2025

## 1. Document Header

*(Included above)*

## 2. Component Architecture

The frontend application will be built using a component-based architecture, likely leveraging a modern JavaScript framework like React, Vue, or Angular. For this guide, we will assume a **React** implementation using functional components and hooks.

Key Components and their relationships:

*   **App (Root Component):** The main entry point. Handles routing and potentially application-wide context providers (e.g., AuthContext, StateContext).
*   **Layout/Shell Components:**
    *   `Header`: Application title, navigation links, user avatar/profile link.
    *   `Sidebar` (Optional): Alternative navigation structure.
    *   `Footer`: Copyright, links.
    *   `PageContainer`: Wraps page content, potentially adding padding or structural elements.
*   **Page Components (Views):** Represent distinct application screens.
    *   `LoginPage`: Handles user authentication (login/signup forms).
    *   `DashboardPage`: Overview of the user's account, twin status, quick links.
    *   `FacebookConnectPage`: Guides the user through connecting their Facebook account.
    *   `TwinCreationStatusPage`: Displays the progress/status of the digital twin creation.
    *   **`ConversationPage` (Core Interaction):** The main interface for interacting with the digital twin via text or voice.
    *   `SettingsPage`: User profile settings, account management, data privacy options.
*   **Feature Components:** Specific pieces of UI within pages.
    *   `AuthForm`: Reusable component for login/signup fields and buttons.
    *   `FacebookConnectButton`: Button to initiate the OAuth flow.
    *   `ProgressIndicator`: Visual feedback for twin creation progress.
    *   `MessageList`: Displays the conversation history in the `ConversationPage`.
    *   `MessageBubble`: Renders a single message (user or twin).
    *   `ChatInput`: Text input area and send button for text chat.
    *   **`VoiceInput`:** Component handling audio recording, visualization, and sending for voice interaction.
    *   `AudioPlayer`: Plays audio responses from the digital twin.
    *   `LoadingSpinner`: Generic loading indicator.
    *   `ErrorDisplay`: Shows error messages.
*   **Reusable/Utility Components:**
    *   `Button`
    *   `Input`
    *   `Modal`
    *   `Avatar`

**Relationship Hierarchy (Conceptual):**

```
App
├── Layout (Header, Footer, PageContainer)
│   └── Router
│       ├── LoginPage
│       ├── DashboardPage
│       ├── FacebookConnectPage
│       ├── TwinCreationStatusPage
│       │   └── ProgressIndicator
│       ├── ConversationPage
│       │   ├── MessageList
│       │   │   └── MessageBubble (User & Twin variants)
│       │   ├── ChatInput
│       │   ├── VoiceInput
│       │   ├── AudioPlayer
│       │   └── LoadingSpinner (while waiting for twin response)
│       └── SettingsPage
├── Shared Components (Button, Input, Modal, etc. - used across pages)
└── Context Providers (AuthContext, etc.)
```

## 3. State Management

Effective state management is crucial for a dynamic application like this, handling user authentication, twin status, conversation history, and UI states (loading, errors, modals).

**Approaches:**

1.  **Local Component State (`useState`, `useReducer`):** Suitable for managing state within a single component (e.g., input field value, button disabled state, modal open/closed).
2.  **Context API (`useContext`):** Good for sharing state (like user authentication status, theme) among components without prop drilling, but can lead to performance issues if context updates frequently and many components consume it.
3.  **Dedicated State Management Library:** Recommended for complex global state, such as:
    *   User Authentication and Profile Data
    *   Facebook Connection Status
    *   Digital Twin Status and Configuration
    *   Conversation History (can grow large)
    *   Application-wide loading/error states

**Recommendation:**

Use **Zustand** or **Redux Toolkit** for managing global application state. Zustand is often preferred for its simplicity and less boilerplate, while Redux Toolkit provides a more structured approach with powerful features like RTK Query (excellent for handling server state via API calls, which often overlaps with application state).

For managing server-cached data resulting from API calls (like conversation history, twin status), a library like **React Query (TanStack Query)** is highly recommended. It simplifies fetching, caching, invalidating, and synchronizing server state, significantly reducing boilerplate and improving performance compared to manual `useEffect` logic.

**Example State Needs:**

*   `auth.isLoggedIn: boolean`
*   `auth.user: User | null`
*   `facebook.isConnected: boolean`
*   `twin.status: 'idle' | 'creating' | 'ready' | 'error'`
*   `twin.creationProgress: number | null`
*   `conversation.messages: Message[]`
*   `conversation.isLoadingResponse: boolean`
*   `ui.isModalOpen: boolean`
*   `app.error: string | null`

## 4. UI Design

The UI needs to be intuitive, guiding users through the onboarding process (connecting Facebook, twin creation) and providing a seamless experience for interacting with their digital twin.

**Key Layout Considerations:**

*   **Responsive Design:** The application must work well on desktop browsers, tablets, and mobile phones, especially the core conversation interface.
*   **Clear Onboarding Flow:** A multi-step process with clear instructions, progress indicators, and feedback during Facebook connection and twin creation.
*   **Conversation Interface:**
    *   Layout similar to standard messaging apps (message bubbles, input area at the bottom).
    *   Distinct styling for user messages and twin messages.
    *   Avatar or visual indicator for the digital twin.
    *   Input area should support both text input and a prominent button/area for initiating voice input.
    *   Visual feedback when the twin is processing a request (e.g., typing indicator, loading spinner).
    *   Handle long conversation histories (virtualization if needed).
*   **Status Indicators:** Clearly show the status of the twin (e.g., "Creating...", "Ready to chat").
*   **Error Handling:** Display clear and user-friendly error messages for failed API calls, Facebook connection issues, etc.
*   **Accessibility:** Ensure keyboard navigation, ARIA attributes, and sufficient color contrast.

**Key User Interactions:**

*   **Authentication:** Standard login/signup forms.
*   **Facebook Connection:** Button initiating OAuth redirect, handling redirect back to the app.
*   **Text Chat:** Typing in an input field, clicking "Send" or pressing Enter.
*   **Voice Chat:**
    *   Clicking a microphone icon to start recording.
    *   Visual feedback during recording (e.g., audio waveform visualization).
    *   Clicking the icon again or a separate button to stop recording and send the audio.
    *   Playing back the digital twin's audio response.
*   **Navigation:** Clear navigation between different sections (Dashboard, Conversation, Settings).

## 5. API Integration

The frontend communicates with the backend API to perform actions and fetch data.

**Key API Endpoints (Examples):**

*   `POST /api/auth/login`: User login.
*   `POST /api/auth/signup`: User registration.
*   `GET /api/user`: Fetch current user details.
*   `POST /api/facebook/connect`: Initiate Facebook OAuth flow (backend handles redirect).
*   `GET /api/facebook/status`: Check if Facebook is connected and data ingestion status.
*   `GET /api/twin/status`: Get the digital twin creation status for the user.
*   `GET /api/twin/{twinId}/messages`: Fetch conversation history.
*   `POST /api/twin/{twinId}/message` (Text): Send a text message to the twin.
    *   Request Body: `{ "text": "Hello twin!" }`
    *   Response Body: `{ "response": "Hello back!", "type": "text" }`
*   `POST /api/twin/{twinId}/voice` (Voice): Send audio data to the twin.
    *   Request Body: Binary audio data (e.g., WAV, OGG)
    *   Response Body: Binary audio data (e.g., MP3, OGG) and potentially a text transcript.
*   `GET /api/twin/{twinId}/voice/transcript/{messageId}` (Optional): Fetch transcript if needed separately.

**Implementation:**

*   Use a library like **Axios** or the built-in `fetch` API for making HTTP requests. Axios offers advantages like interceptors for adding auth headers automatically.
*   Manage authentication tokens (e.g., JWT) received upon login. Store them securely (e.g., in `localStorage` or `sessionStorage`, though consider security implications). Include the token in the `Authorization: Bearer <token>` header for authenticated requests.
*   Handle different HTTP status codes (200 OK, 201 Created, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Internal Server Error). Display appropriate error messages to the user.
*   Use **React Query** to manage the state of API calls. It handles loading states, errors, caching, background refetching, and more, significantly cleaning up data fetching logic within components.

## 6. Testing Approach

A robust testing strategy is essential to ensure the quality and stability of the frontend application.

**Types of Tests:**

1.  **Unit Tests:** Test individual functions, components in isolation.
    *   **Tools:** Jest (test runner), React Testing Library (for testing React components focusing on user interaction).
    *   **What to test:** Pure functions, small utility modules, component rendering with different props, basic component interactions (e.g., button clicks, input changes) without complex state or API dependencies.
2.  **Integration Tests:** Test how components or modules work together.
    *   **Tools:** Jest, React Testing Library.
    *   **What to test:** Interaction between a component and its child components, testing hooks, testing a small flow (e.g., a form submission handled by an API hook). Mock API calls for these tests.
3.  **End-to-End (E2E) Tests:** Test full user flows through the application in a real browser environment.
    *   **Tools:** Cypress, Playwright.
    *   **What to test:** User registration/login flow, connecting Facebook, navigating to the conversation page and sending a message (text and voice), checking if the response is displayed correctly.

**Testing Strategy:**

*   **Prioritize Critical Paths:** Focus testing efforts on the core user flows: authentication, connecting Facebook, and the digital twin conversation.
*   **Component Testing (React Testing Library):** Aim for good coverage of your key components, especially interactive ones, testing how they render and respond to user events. Test components from the user's perspective (what they see and interact with).
*   **Hook Testing:** If using custom hooks (e.g., for API calls, state logic), test their behavior independently.
*   **Mocking:** Use Jest mocks to isolate units during testing (e.g., mock API calls, mock browser APIs like `fetch`, `MediaRecorder`).
*   **Test Environment:** Set up a consistent testing environment (e.g., using Jest's configuration).
*   **Automate:** Integrate tests into your CI/CD pipeline.

## 7. Code Examples

Here are sample code snippets illustrating key concepts. Assumes React with TypeScript, Axios, and React Query.

**Example 1: MessageBubble Component (Presentation)**

```typescript
// src/components/MessageBubble.tsx
import React from 'react';
import './MessageBubble.css'; // Assuming CSS for styling

interface MessageBubbleProps {
  text?: string; // Optional for voice messages
  audioUrl?: string; // Optional for voice messages
  isUserMessage: boolean;
}

const MessageBubble: React.FC<MessageBubbleProps> = ({ text, audioUrl, isUserMessage }) => {
  const bubbleClass = isUserMessage ? 'message-bubble user' : 'message-bubble twin';

  return (
    <div className={bubbleClass}>
      {text && <p>{text}</p>}
      {audioUrl && (
        <audio controls src={audioUrl}>
          Your browser does not support the audio element.
        </audio>
      )}
    </div>
  );
};

export default MessageBubble;

/*
// Example CSS (src/components/MessageBubble.css)
.message-bubble {
  max-width: 70%;
  padding: 10px 15px;
  border-radius: 18px;
  margin-bottom: 10px;
  word-wrap: break-word;
}

.message-bubble p {
  margin: 0;
}

.message-bubble audio {
    width: 100%; /* Or adjust as needed */
    max-width: 200px; /* Prevent audio player from being too wide */
}


.message-bubble.user {
  background-color: #007bff;
  color: white;
  margin-left: auto; /* Align right */
  border-bottom-right-radius: 2px;
}

.message-bubble.twin {
  background-color: #e9e9eb;
  color: #333;
  margin-right: auto; /* Align left */
  border-bottom-left-radius: 2px;
}
*/
```

**Example 2: Using React Query to Fetch Conversation History**

```typescript
// src/hooks/useConversation.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import axios from 'axios';

interface Message {
  id: string;
  text?: string;
  audioUrl?: string; // URL to fetch audio from backend
  isUserMessage: boolean;
  timestamp: string;
}

interface SendMessagePayload {
  twinId: string;
  text?: string;
  audioBlob?: Blob; // Use Blob for voice data
}

const fetchConversationMessages = async (twinId: string): Promise<Message[]> => {
  const { data } = await axios.get<Message[]>(`/api/twin/${twinId}/messages`, {
    headers: { Authorization: `Bearer ${localStorage.getItem('authToken')}` }, // Example auth header
  });
  return data;
};

const sendMessage = async ({ twinId, text, audioBlob }: SendMessagePayload): Promise<Message> => {
  const headers = { Authorization: `Bearer ${localStorage.getItem('authToken')}` }; // Example auth header

  if (audioBlob) {
    const formData = new FormData();
    formData.append('audio', audioBlob, 'voice_message.wav'); // Backend expects 'audio' field
    const { data } = await axios.post<Message>(`/api/twin/${twinId}/voice`, formData, { headers });
    return data;
  } else if (text) {
    const { data } = await axios.post<Message>(`/api/twin/${twinId}/message`, { text }, { headers });
    return data;
  }
  throw new Error("Cannot send an empty message.");
};

export const useConversation = (twinId: string) => {
  const queryClient = useQueryClient();

  // Fetch conversation history
  const { data: messages, isLoading, error } = useQuery<Message[], Error>(
    ['conversation', twinId], // Query key
    () => fetchConversationMessages(twinId),
    {
      enabled: !!twinId, // Only run query if twinId exists
      staleTime: 5 * 60 * 1000, // Data considered "fresh" for 5 minutes
      cacheTime: 10 * 60 * 1000, // Data kept in cache for 10 minutes
      // onSuccess: (data) => { /* Optional side effects */ }
    }
  );

  // Mutation for sending messages
  const { mutate: sendTextMessage, isLoading: isSendingText } = useMutation<Message, Error, string>(
    (text) => sendMessage({ twinId, text }),
    {
      onSuccess: (newMessage) => {
        // Optimistically update the cache or refetch after success
        queryClient.setQueryData<Message[]>(['conversation', twinId], (oldMessages) => {
            // Append the user's message immediately (optional optimistic update)
             const userMessage: Message = {
                id: `temp-${Date.now()}`, // Temp ID
                text: newMessage.text, // Use text from response for confirmation
                isUserMessage: true,
                timestamp: new Date().toISOString(),
             };
            // Append the twin's response
           return [...(oldMessages || []), userMessage, newMessage]; // Assuming newMessage contains twin's response
        });
        // Or simply invalidate to refetch: queryClient.invalidateQueries(['conversation', twinId]);
      },
      onError: (err) => {
        console.error("Failed to send message:", err);
        // Display error to user
      },
    }
  );

   const { mutate: sendVoiceMessage, isLoading: isSendingVoice } = useMutation<Message, Error, Blob>(
    (audioBlob) => sendMessage({ twinId, audioBlob }),
    {
      onSuccess: (newMessage) => {
         // Optimistically update or refetch as above
         queryClient.invalidateQueries(['conversation', twinId]); // Invalidate is simpler for voice for now
      },
      onError: (err) => {
        console.error("Failed to send voice message:", err);
         // Display error to user
      },
    }
  );


  return {
    messages: messages || [], // Provide empty array if data is null
    isLoading: isLoading || isSendingText || isSendingVoice,
    error,
    sendTextMessage,
    sendVoiceMessage,
  };
};
```

**Example 3: Basic Chat Input Component**

```typescript
// src/components/ChatInput.tsx
import React, { useState, useRef } from 'react';
import { useConversation } from '../hooks/useConversation'; // Using the hook from Example 2
import './ChatInput.css'; // Styling
// Assume you have a VoiceRecorder hook/component for actual recording

interface ChatInputProps {
  twinId: string;
}

const ChatInput: React.FC<ChatInputProps> = ({ twinId }) => {
  const [textInput, setTextInput] = useState('');
  const { sendTextMessage, isLoading: isSendingText } = useConversation(twinId);
  // You would integrate your voice recording logic here
  // const { startRecording, stopRecording, isRecording, audioBlob } = useVoiceRecorder();
  // const { sendVoiceMessage, isLoading: isSendingVoice } = useConversation(twinId);

  const inputRef = useRef<HTMLInputElement>(null);

  const handleSendText = () => {
    if (textInput.trim() && !isSendingText) {
      sendTextMessage(textInput);
      setTextInput(''); // Clear input after sending
      inputRef.current?.focus(); // Keep focus on input
    }
  };

  const handleKeyPress = (event: React.KeyboardEvent<HTMLInputElement>) => {
    if (event.key === 'Enter') {
      event.preventDefault(); // Prevent new line in input
      handleSendText();
    }
  };

  // Basic structure for voice input integration (logic omitted)
  // const handleVoiceButtonClick = () => {
  //    if (isRecording) {
  //        stopRecording();
  //        // After stopRecording, audioBlob hook should be populated,
  //        // you'd then call sendVoiceMessage(audioBlob)
  //    } else {
  //        startRecording();
  //    }
  // }

  // useEffect(() => {
  //    if (audioBlob && !isSendingVoice) {
  //       sendVoiceMessage(audioBlob);
  //    }
  // }, [audioBlob, isSendingVoice, sendVoiceMessage]);


  return (
    <div className="chat-input-container">
      <input
        ref={inputRef}
        type="text"
        value={textInput}
        onChange={(e) => setTextInput(e.target.value)}
        onKeyPress={handleKeyPress}
        placeholder="Type a message..."
        disabled={isSendingText /* || isSendingVoice */}
      />
      {/* {isRecording ? (
          <button onClick={handleVoiceButtonClick} disabled={isSendingText || isSendingVoice}>
             ⏹ Stop Recording
          </button>
      ) : (
           <button onClick={handleVoiceButtonClick} disabled={isSendingText || isSendingVoice}>
             🎙 Start Recording
          </button>
      )} */}
      <button onClick={handleSendText} disabled={isSendingText || !textInput.trim()}>
        Send
      </button>
    </div>
  );
};

export default ChatInput;

/*
// Example CSS (src/components/ChatInput.css)
.chat-input-container {
    display: flex;
    padding: 10px;
    background-color: #f8f9fa;
    border-top: 1px solid #dee2e6;
    align-items: center;
}

.chat-input-container input {
    flex-grow: 1;
    padding: 10px;
    border: 1px solid #ced4da;
    border-radius: 20px;
    margin-right: 10px;
    font-size: 1rem;
    outline: none;
}

.chat-input-container input:focus {
    border-color: #007bff;
    box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.chat-input-container button {
    padding: 10px 20px;
    border: none;
    border-radius: 20px;
    background-color: #007bff;
    color: white;
    cursor: pointer;
    font-size: 1rem;
    transition: background-color 0.2s ease;
}

.chat-input-container button:hover:not(:disabled) {
    background-color: #0056b3;
}

.chat-input-container button:disabled {
    background-color: #a0c7ff;
    cursor: not-allowed;
}
*/
```

This guide provides a foundational structure and highlights key considerations for building the frontend of your AI Digital Twin SaaS. Remember to adapt these principles to your specific technology choices and project requirements.
```
