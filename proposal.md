## Feature Proposal: Enhanced Prefetching for Phoenix LiveView

### Overview

This proposal is an advanced, optional prefetching system in Phoenix LiveView to optimize dynamic navigation responsiveness and user experience. 

---

### How It Works

Prefetching is controlled by a `prefetch` attribute on `.link` components, which can have the following values:

- **`"viewport"`**: Prefetches the view when the link becomes visible in the viewport.
- **`"hover"`**: Prefetches the view when the mouse hovers over the link, for at least a certain time.
- **`"mouse-down"`**: Prefetching begins when the user presses down on a button (mouse down event).
- **`"none"`**: Prefetching is disabled for the link. It is useful if prefetching is set from a parent layout.

The syntax for defining prefetching in `.html.heex` files is as follows:

```html
<.link prefetch="..." max-age="10s/min/h or max">
```

- The presence of the `prefetch` attribute indicates that prefetching is enabled unless explicitly set to `"none"`.
- The `max-age` attribute defines the duration for which the prefetch cache remains valid. Once invalidated, prefetching is re-triggered based on conditions like hover or visibility. The default value is 30s.


Prefetching can also be configured from layouts, using attributes. This way, it's possible to set a default prefetching policy for all views that inherit from a certain layout (like admin dashboard), which can later be overridden by individual link components.

---


### State Management
When the server responds with a prefetched view, it stores all of the assigns related to that view inside the current assigns collection, differentiated by the route that was prefetched. Here's a high-level example:

```
socket -> assigns -> prefetch -> "/get_cars?page=5 @ 21:05:30... (The expiry timestamp)" -> cars -> [...]
```

This approach ensures that the state of prefetches will not be recalculated. Also, using this hierarchical structure for preserving assigns couples them with the current socket, so they'll be cleaned up when the session is ended.

### Server-Client Interaction

The server and client coordinate through LiveView's WebSocket connection to handle prefetching requests. When the client sends a prefetch request, the server sends the view, which is cached by the client in the browser session storage. This will minimize the client-side logic for cache invalidation because the browser will clear session storage once the session is ended. Upon user interaction, the cached view is rendered immediately, providing a seamless experience, even on mid/high-latency connections.

---

### Rendering Logic for Prefetching

During prefetching, the system renders the view with all elements and logic except for asynchronous functions (e.g., `assign_async`, `start_async`). This behavior is controlled by a special bit sent from the client to the server, indicating whether the request is a prefetch or a real navigation. If it is unset, it means that a full-on interactive navigation is requested (this decision ensures maximum compatibility with previous versions of LiveView). If it's 0, it means that the prefetching is requested, so async calls shouldn't be made. And if it's set to 1, the server has to only make async calls and return necessary DOM patches to the client, as usual. The huge benefit of this implementation is that it utilizes the abstractions that are already present in the framework, so this proposal can be implemented with the least possible additions.

#### Client-Side Rate Limiting
The Client-Side implementation should have a simple rate-limiting logic where it limits the amount of prefetch requests the client sends in a second. This approach can time-distribute accidental overloads caused by the large number of prefetchable views, and putting requests in a queue so they don't get rejected either.

---

### Client-Side Implementation

The client-side implementation minimizes complexity and adheres to LiveView standards. Prefetching logic can be bundled into a separate, optional file, allowing projects that do not require prefetching to exclude it entirely for simplicity and performance optimization. It has to handle the custom HTML attributes introduced by this feature, and the prefetch-and-cache logic.

---

### Prefetch Cache Management

To enhance control, a function will be introduced to clear the prefetch cache for the entire site. This is particularly useful in scenarios like user logout, ensuring sensitive data is not inadvertently cached.

---

## FAQs

- **Are the gains of making a whole separate JS file for the client, and adding code at the top of the current system backend worth the benefits of prefetching?**
Definitely. Take the official example of Phoenix LiveView, [LiveBeats](https://livebeats.fly.dev/). One of the most noticeable things about this app is that it doesn't feel as responsive as some other SPAs you've worked with. The reason lies behind the foundation of Phoenix LiveView. It uses WebSocket connections to communicate with client, eliminating the need of having client-side code to achieve the same result. While this approach brings significant advantages over giving the recipe to the client and "hoping they cook well", it introduces a big problem: **latency**. This effect can significantly impact on web application user experience, especially if the end-user has a poor connection, or is geographically distant from where the servers are located. The simplest solution to the latency problem is to load things before they're requested, so they get immediately swapped, providing a much snappier UX.

- **Doesn't prefetching every link on the screen cause server overload?** No. Prefetching, in the way that was proposed above, won't cause any severe performance bottlenecks. Unless the resource-intensive operations are wrapped inside loading APIs properly, they won't be executed unless the user has made the real navigation. Even some other frameworks, including [Next.js](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#2-prefetching:~:text=Routes%20are%20automatically%20prefetched%20as%20they%20become%20visible%20in%20the%20user%27s%20viewport.) have turned this option on by default. Also, it worth mentioning that this proposal's prefetching implementation is more flexible than many other alternatives.

- **So you mean that preserving the assigns of all pages that could be possibly loaded is a good idea?** While this approach is not the most optimal way to store the data that is coupled to each prefetched view, is has a high potential for performance improvements; especially, in responses to how the assigns cache is cleared, and how is it stored. TODO.

- **What if a link to a non-live component is set to be prefetched?** When the client sends the prefetch request to the server, the server simply responds with a special message, indicating that this route should be loaded using a hard refresh.

- **Who will benefit from this feature?** There's no boundary set for which types of web apps can utilize this feature. If you use Phoenix LiveView's standard APIs to handle loadings, prefetching can be implemented for your project, as simple as adding a script and pasting an attribute to your app's layout.

- **What happens to the projects that don't want to use prefetching?** Nothing, period. This feature will have zero breaking changes, so even if you don't want to use prefetching, nothing stops you from using the latest version of Phoenix LiveView.
