<p align="center">
  <img src="docs/images/logo-full.svg" alt="xplorr" width="300"/>
</p>

<h1 align="center">xplorr</h1>

<p align="center">
  <strong>When you're nearby but nowhere to go</strong> — Let chance decide your next adventure
</p>

<p align="center">
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License: MIT"/></a>
  <img src="https://img.shields.io/badge/Status-In%20Development-blue" alt="Status"/>
  <img src="https://img.shields.io/badge/Domain-xplorr.app-green" alt="Domain"/>
</p>

---

## 🤔 The Problem

You're out in the city. You've got time to kill. There are dozens of places around you, but you're stuck with decision paralysis. "Where should I go?" "What should I do?" Sound familiar?

**xplorr** solves this by taking the decision-making burden off your shoulders. Tap, explore, and let serendipity guide your day.

---

## 💡 What is xplorr?

xplorr is a location-based discovery app that randomly selects places for you to visit based on your vicinity. No more endless scrolling through reviews or second-guessing your choices. Just pure spontaneity with a touch of intelligence.

Perfect for:
- 🚶 Locals looking to rediscover their city
- ✈️ Travelers who want authentic experiences
- 🎯 Anyone suffering from "too many options" syndrome
- 🎲 Spontaneous souls who trust the universe

---

## ✨ Features

### 🎬 Dynamic Splash Screen
The iconic X logo greets you and rotates during exploration

![Splash Screen](docs/images/00-splash-loading.svg)

### 🗺️ Interactive Map View
See all nearby places at a glance with an intuitive map interface. Tap the center X logo to start exploring!

![Home Map View](docs/images/01-home-map-view.svg)

### 🎯 Route Randomizer
Watch as animated routes pulse from your location to randomly selected destinations. The X logo rotates as the algorithm chooses your next adventure.

![Route Randomizer](docs/images/02-roulette-spinner.svg)

### 📍 Smart Place Details
Get all the info you need: distance, ratings, estimated visit time, and more

![Place Details](docs/images/03-place-details.svg)

### 📋 Dynamic Itinerary Builder
Build a full day's adventure with multiple randomly discovered spots

![Itinerary View](docs/images/04-itinerary-view.svg)

### ⚙️ Privacy-First Settings
Control your location sharing and customize your discovery preferences

![Settings Screen](docs/images/05-settings-screen.svg)

---

## 🚀 Key Capabilities

- **Smart Randomization**: Never visit the same place twice (unless you want to)
- **No-Go Zones**: Mark places you don't want to see in future spins
- **Visit History**: Track your adventures automatically
- **Location Control**: Use real-time GPS or set a fixed location
- **Itinerary Builder**: Chain multiple places into a full day plan
- **Tap-to-Explore**: Simple tap interaction for seamless discovery
- **Offline-Ready**: Once loaded, your nearby places work without connectivity

---

## 🎯 How It Works

1. **Share Your Location** (or set a fixed one in settings)
2. **View Nearby Places** on an interactive map
3. **Tap the X logo** at your location to start exploring
4. **Watch the animation** as routes pulse to random destinations
5. **Discover** your randomly selected place
6. **Add to itinerary** or **explore again**
7. **Build** a full day's adventure with multiple places
8. **Go!** Start your spontaneous journey

### The Algorithm

```
1. User taps X logo → Trigger randomization
2. X logo rotates, map animates routes
3. Fetch places within radius (configurable)
4. Filter out:
   - Already visited places
   - No-go tagged locations
   - Currently closed venues (optional)
5. Weighted random selection based on:
   - Distance (closer = slightly higher weight)
   - Category diversity (avoid clustering)
   - Ratings (minimum threshold)
6. Animate route pulse → Present result
7. Store selection in history
```

---

## 🛠️ Tech Stack (Planned)

### Frontend
- **React Native** / **Flutter** — Cross-platform mobile development
- **MapKit** / **Google Maps SDK** — Interactive map rendering
- **Reanimated** — Smooth gesture animations

### Backend
- **Node.js** + **Express** / **Firebase** — API and authentication
- **PostgreSQL** / **MongoDB** — User data and preferences
- **Redis** — Session management and caching

### APIs & Services
- **Google Places API** / **Foursquare API** — Place data
- **Geolocation API** — User positioning
- **OpenStreetMap** (alternative) — Free mapping data

---

## 📱 User Flow

```
┌─────────────────┐
│   Open App      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐      ┌──────────────────┐
│ Location Prompt │─────▶│  Grant Access    │
└────────┬────────┘      └────────┬─────────┘
         │                        │
         ▼                        │
┌─────────────────┐              │
│   Map View      │◀─────────────┘
│  (12 places)    │
└────────┬────────┘
         │
         ▼ (Tap X Logo)
┌─────────────────┐
│ Route Animation │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Place Details   │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌────────┐  ┌──────────┐
│Add to  │  │Explore   │
│Trip    │  │          │
└───┬────┘  └─────┬────┘
    │             │
    ▼             └─────────────┐
┌─────────────────┐             │
│   Itinerary     │             │
│   Builder       │◀────────────┘
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Start Trip     │
└─────────────────┘
```

---

## 🔒 Privacy & Data

- **Location Data**: Stored locally, only sent to server for place queries
- **Visit History**: Encrypted and stored per device
- **No-Go List**: Private, never shared
- **Analytics**: Opt-in only, anonymous aggregated data
- **Data Deletion**: One-tap complete data wipe in settings

---

## 🎨 Design Philosophy

- **Minimal Friction**: Get to discovery in just 1 tap
- **Visual Feedback**: Animated routes and rotating X logo
- **Map-First**: Full-screen map with floating UI elements
- **Trust the Randomness**: Embrace uncertainty as a feature
- **Personality**: Playful, exploratory, adventurous

---

## 🗓️ Roadmap

### Phase 1: MVP (Current)
- [x] Core UI mockups
- [ ] Map integration
- [ ] Basic randomization algorithm
- [ ] Location services
- [ ] Single place discovery

### Phase 2: Enhanced Discovery
- [ ] Multi-place itinerary builder
- [ ] Visit history tracking
- [ ] No-go list functionality
- [ ] Place categories filter
- [ ] Adjustable search radius

### Phase 3: Social & Smart
- [ ] Share itineraries with friends
- [ ] Group discovery (multiple users, one spin)
- [ ] ML-based preferences (learn what you like)
- [ ] Weather-aware suggestions
- [ ] Time-of-day recommendations

### Phase 4: Gamification
- [ ] Achievement badges
- [ ] Discovery streaks
- [ ] Local explorer leaderboards
- [ ] Community challenges

---

## 🤝 Contributing

This project is in early development. Contributions, ideas, and feedback are welcome!

### Setup (Coming Soon)
```bash
# Clone the repository
git clone https://github.com/yourusername/xplorr.git

# Install dependencies
npm install

# Run development server
npm run dev
```

---

## 📄 License

MIT License - feel free to use this concept for your own projects!

---

## 🎯 Why "xplorr"?

The name combines "explore" with the spirit of discovery. The double 'r' gives it a modern, tech-forward feel, while the X logo with arrowheads pointing in all directions perfectly represents the randomness and adventure of discovering new places. Plus, **xplorr.app** was available! 🎉

### The X Logo

The iconic X with arrowheads pointing outward symbolizes:
- **Exploration** in all directions
- **Discovery** without predetermined paths
- **Randomness** - every direction is a possibility
- **Connection** between you and nearby places

<p align="center">
  <img src="docs/images/logo-icon.svg" alt="xplorr X logo" width="100"/>
</p>

---

## 💬 Contact & Feedback

Have ideas? Found a bug? Want to collaborate?

- 📧 Email: hello@xplorr.app (coming soon)
- 🐦 Twitter: @xplorrapp (coming soon)
- 💬 Discord: [Join our community](https://discord.gg/xplorr) (coming soon)
- 🌐 Website: https://xplorr.app (coming soon)

---

**Remember**: The best adventures are the ones you don't plan. Let xplorr be your guide. 🎯


