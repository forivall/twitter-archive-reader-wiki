This page suppose you already obtain a `Conversation` object, through the `DMArchive` instance (see [Browsing Direct Message archive](./Browsing-Direct-Message-archive-(conversations))).

`Conversation` object will be named, for convention, `conversation`.

Note that some filtering methods returns a `SubConversation` object, that expose the same public methods that `Conversation` (except conversation ID `.id`).

`Conversation` give access to included some `LinkedDirectMessage`, that hold the data of one message.
Detail of the interface is given at the end of this page.

- `Conversation.all: LinkedDirectMessage[]`

Basic access to every direct message stored in current `Conversation`.

- `Conversation.find(query: RegExp): SubConversation`

Find direct messages using a query. Return a filtered conversation.

```ts
// Search for messages having specific text (use a RegExp to validate)
conversation.find(/Hello !/i);
```

- `Conversation.month(month: string, year: string): SubConversation`

Get a subconversation containing all the messages of a specific month.

- `Conversation.sender(ids: string | string[]): SubConversation`

Find direct messages sender by a specific user ID. Return a filtered conversation.

- `Conversation.recipient(ids: string | string[]): SubConversation`

Find direct messages recieved by a specific user ID. Return a filtered conversation.

- `Conversation.between(since: Date, until: Date): SubConversation`

Find direct messages send between two dates. Return a filtered conversation.

```ts
// Search for messages sent in a specific date
conversation.between(new Date("2019-01-01"), new Date("2019-02-04"));
```

- `Conversation.around(id: string, context?: number)`

Find context around a direct message. Returns a object containing the n-before and the n-after messages asked with *context*.

```ts
conversation.around("19472928432");

=> {
  before: LinkedDirectMessage[],
  current: LinkedDirectMessage,
  after: LinkedDirectMessage[]
}
```

- `Symbol.iterator`

Instances of `Conversation` are iterable.

```ts
for (const message of conversation) {
  // message fulfill LinkedDirectMessage interface
}
```

- Chaining

You can chain methods that returns `SubConversation` objects.

```ts
// Every truncature method [.find(), .between(), .month(), .sender(), .recipient()]
// returns a sub-object (SubConversation) that have his own index and own methods.
// This allow you to chain methods:
conversation
  .find(/Hello/i)
  .between(new Date("2019-01-01"), new Date("2019-02-01"))
  .recipient(["MY_USER_1", "MY_USER_2"]);
```

- `Conversation.index: ConversationIndex`

Get the conversation details, with messages sorted by year, month and day.

- `Conversation.length: number`

Number of messages in this conversation.

- `Conversation.participants: Set<string>`

User IDs of the participants of this conversation.

- `Conversation.is_group_conversation: boolean`

True if the conversation is a group conversation.

- `Conversation.first: LinkedDirectMessage`

First DM in the conversation.

- `Conversation.last: LinkedDirectMessage`

Last DM in the conversation.


## Direct message data

Here's the properties available in a `DirectMessage`.
```ts
interface DirectMessage {
  /** Person who get the DM (Twitter user ID). */
  recipientId: string;
  /** Content of the DM. */
  text: string;
  /** 
   * Array of URLs linked to this direct message. 
   * Currently, a DM could only contain **one** media. 
   */
  mediaUrls: string[];
  /** Person who send the DM (Twitter user ID). */
  senderId: string;
  /** Message ID. */
  id: string;
  /** Stringified date of message creation. 
   * If the DM is a `LinkedDirectMessage`, 
   * please use **.createdAtDate** property to get the date,
   * it's already correctly parsed. 
   */
  createdAt: string;
}
```

Interface `LinkedDirectMessage`, that validate DMs in `Conversation` objects, add a `.previous` and `.next` property, linking the following and previous DM in the current conversation.

To get medias linked in one message, please see [Dealing with medias](./Dealing-with-medias.md).

## Direct message events

Events are things created in group conversations in order to reflect things happenning in a discussion that are **not** messages.
It includes conversation name change, participant join and leave, etc.

You can see events that happen **before** and **after** a single message with properties `.events.before` and `.events.after`.
If any event exists after or before a DM, property `.events` will not be defined.

In `.events.before` and `.events.after`, events are grouped by categories.
See interface `DirectMessageEventsContainer` in `types/GDPRDMs.ts` file in order to see them and have a description of every available property.

```ts
const conv = archive.messages.all[0];

const msg = conv.single('dm_id');
if (msg.events) {
  if (msg.events.before) {
    console.log(
      "The followings events happended before the DM:", 
      Object.entries(msg.events.before)
        .map(([ev_type, events]) => `${ev_type} (${events.length} times)`)
        .join(', ')
    )
  }
  if (msg.events.after) {
    console.log(
      "The followings events happended after the DM:", 
      Object.entries(msg.events.after)
        .map(([ev_type, events]) => `${ev_type} (${events.length} times)`)
        .join(', ')
    )
  }
}
```

---

You can also iterate over events, via helper `TwitterHelpers.getEventsFromMessages()` or via `Conversation.events()` method.

```ts
const conversation = archive.messages.all[0];

// Get all events without messages...
TwitterHelpers.getEventsFromMessages(conversation.all, /* include_messages = false */);
// ...it is the same as
conversation.events(/* include_messages = false */);
```

A single event is structured as:
```ts
event === {
  [eventName]: {
    createdAt: string, 
    createdAtDate: Date,
    ...(event properties)
  }
}

// For example, here's a direct message event representating a message
event = {
  messageCreate: {
    createdAt: 'xxx',
    createdAtDate: Date,
    text: 'xxx',
    senderId: 'xxx',
    recipientId?: 'xxx',
    mediaUrls: ['xxx'],
    id: 'xxx',
    previous: null | LinkedDirectMessage,
    next: null |  LinkedDirectMessage
  }
}
```


## Continue

Next part is [Dealing with medias](./Dealing-with-medias).

