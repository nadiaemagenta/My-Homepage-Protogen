# Tech Brief: Open-Meteo Weather Widget Implementation

## Overview
We'll integrate Open-Meteo's free weather API into your static HTML/CSS website on Vercel to display real-time weather information. Open-Meteo is ideal for this use case as it requires no API key, has no rate limits for reasonable usage, and provides reliable weather data.

## Technical Architecture

### API Service: Open-Meteo
- **Endpoint**: `https://api.open-meteo.com/v1/forecast`
- **Authentication**: None required
- **Rate Limits**: None for reasonable usage
- **Data Format**: JSON
- **CORS**: Enabled (allows direct browser requests)

### Implementation Approach
- **Client-side JavaScript**: Fetch API calls directly from the browser
- **Geolocation**: Optional - can use browser geolocation or hardcoded coordinates
- **Caching**: Browser cache + optional localStorage for offline fallback
- **Error Handling**: Graceful degradation with fallback messages

### Data Structure
Open-Meteo returns structured JSON with current weather and forecast data:
```json
{
  "current_weather": {
    "temperature": 45.2,
    "windspeed": 8.1,
    "weathercode": 3,
    "time": "2026-03-06T21:45"
  }
}
```

## Features to Implement

### Core Features
- Current temperature display
- Weather condition (sunny, cloudy, rainy, etc.)
- Wind speed and direction
- Location display
- Last updated timestamp

### Enhanced Features (Optional)
- 7-day forecast
- Hourly forecast for today
- Weather icons
- User location detection
- Temperature unit toggle (°F/°C)
- Responsive design

---

# Step-by-Step Implementation Plan

## Phase 1: Basic Setup (30 minutes)

### Step 1: Create HTML Structure
Add the weather widget container to your existing HTML:

```html
<!-- Add this where you want the weather widget to appear -->
<div class="weather-widget" id="weather-widget">
  <div class="weather-loading">
    <span>🌤️</span>
    <p>Loading weather data...</p>
  </div>
</div>
```

### Step 2: Add Base CSS Styling
Add these styles to your CSS file:

```css
.weather-widget {
  max-width: 300px;
  padding: 20px;
  border-radius: 12px;
  background: linear-gradient(135deg, #74b9ff, #0984e3);
  color: white;
  font-family: Arial, sans-serif;
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
}

.weather-loading {
  text-align: center;
}

.weather-loading span {
  font-size: 2em;
  display: block;
  margin-bottom: 10px;
}

.weather-info {
  text-align: center;
}

.weather-temp {
  font-size: 3em;
  font-weight: bold;
  margin: 10px 0;
}

.weather-condition {
  font-size: 1.2em;
  margin-bottom: 15px;
  opacity: 0.9;
}

.weather-details {
  display: flex;
  justify-content: space-between;
  font-size: 0.9em;
  opacity: 0.8;
}

.weather-error {
  text-align: center;
  color: #ff7675;
  background: #ffe0e0;
  color: #d63031;
  padding: 15px;
  border-radius: 8px;
}
```

## Phase 2: Core JavaScript Implementation (45 minutes)

### Step 3: Create Weather Service
Add this JavaScript to your HTML (before closing `</body>` tag):

```html
<script>
class WeatherWidget {
  constructor(containerId, options = {}) {
    this.container = document.getElementById(containerId);
    this.latitude = options.latitude || 47.6062; // Default: Seattle
    this.longitude = options.longitude || -122.3321;
    this.city = options.city || 'Seattle';
    this.temperatureUnit = options.temperatureUnit || 'fahrenheit';
    
    this.init();
  }

  async init() {
    try {
      await this.fetchWeather();
    } catch (error) {
      this.showError('Unable to load weather data');
    }
  }

  async fetchWeather() {
    const url = `https://api.open-meteo.com/v1/forecast?latitude=${this.latitude}&longitude=${this.longitude}&current_weather=true&temperature_unit=${this.temperatureUnit}&windspeed_unit=mph&timezone=auto`;
    
    const response = await fetch(url);
    if (!response.ok) throw new Error('Weather API request failed');
    
    const data = await response.json();
    this.displayWeather(data);
  }

  displayWeather(data) {
    const current = data.current_weather;
    const condition = this.getWeatherCondition(current.weathercode);
    
    this.container.innerHTML = `
      <div class="weather-info">
        <div class="weather-location">${this.city}</div>
        <div class="weather-icon">${condition.icon}</div>
        <div class="weather-temp">${Math.round(current.temperature)}°${this.temperatureUnit === 'fahrenheit' ? 'F' : 'C'}</div>
        <div class="weather-condition">${condition.description}</div>
        <div class="weather-details">
          <div>Wind: ${current.windspeed} mph</div>
          <div>Updated: ${new Date(current.time).toLocaleTimeString()}</div>
        </div>
      </div>
    `;
  }

  getWeatherCondition(code) {
    const conditions = {
      0: { icon: '☀️', description: 'Clear sky' },
      1: { icon: '🌤️', description: 'Mainly clear' },
      2: { icon: '⛅', description: 'Partly cloudy' },
      3: { icon: '☁️', description: 'Overcast' },
      45: { icon: '🌫️', description: 'Foggy' },
      48: { icon: '🌫️', description: 'Rime fog' },
      51: { icon: '🌦️', description: 'Light drizzle' },
      53: { icon: '🌦️', description: 'Moderate drizzle' },
      55: { icon: '🌧️', description: 'Dense drizzle' },
      61: { icon: '🌧️', description: 'Slight rain' },
      63: { icon: '🌧️', description: 'Moderate rain' },
      65: { icon: '🌧️', description: 'Heavy rain' },
      80: { icon: '🌦️', description: 'Rain showers' },
      95: { icon: '⛈️', description: 'Thunderstorm' }
    };
    
    return conditions[code] || { icon: '🌤️', description: 'Unknown' };
  }

  showError(message) {
    this.container.innerHTML = `
      <div class="weather-error">
        <p>⚠️ ${message}</p>
      </div>
    `;
  }
}

// Initialize the weather widget when page loads
document.addEventListener('DOMContentLoaded', function() {
  new WeatherWidget('weather-widget', {
    city: 'Seattle',
    latitude: 47.6062,
    longitude: -122.3321,
    temperatureUnit: 'fahrenheit'
  });
});
</script>
```

## Phase 3: Enhanced Features (Optional - 60 minutes)

### Step 4: Add Geolocation Support
Replace the initialization code with:

```javascript
document.addEventListener('DOMContentLoaded', function() {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      function(position) {
        new WeatherWidget('weather-widget', {
          latitude: position.coords.latitude,
          longitude: position.coords.longitude,
          city: 'Your Location'
        });
      },
      function(error) {
        // Fallback to default location
        new WeatherWidget('weather-widget', {
          city: 'Seattle',
          latitude: 47.6062,
          longitude: -122.3321
        });
      }
    );
  } else {
    // Fallback for browsers without geolocation
    new WeatherWidget('weather-widget');
  }
});
```

### Step 5: Add Refresh Functionality
Add a refresh button and auto-refresh:

```javascript
// Add this method to the WeatherWidget class
addRefreshButton() {
  const refreshBtn = document.createElement('button');
  refreshBtn.innerHTML = '🔄 Refresh';
  refreshBtn.className = 'weather-refresh';
  refreshBtn.onclick = () => this.fetchWeather();
  this.container.appendChild(refreshBtn);
}

// Add auto-refresh (every 10 minutes)
setAutoRefresh() {
  setInterval(() => {
    this.fetchWeather();
  }, 600000); // 10 minutes
}
```

## Phase 4: Testing & Deployment (15 minutes)

### Step 6: Test Locally
1. Open your HTML file in a browser
2. Check browser console for any errors
3. Verify weather data displays correctly
4. Test with different locations (modify coordinates)

### Step 7: Deploy to Vercel
1. Commit your changes to your repository
2. Push to your connected branch
3. Vercel will automatically deploy
4. Test the live version

## Phase 5: Monitoring & Maintenance

### Step 8: Add Error Logging (Optional)
```javascript
// Add to the catch block in fetchWeather()
console.error('Weather widget error:', error);
// Optional: Send to analytics service
```

### Step 9: Performance Optimization
- Add localStorage caching for offline support
- Implement request debouncing
- Add loading states for better UX

## Timeline Summary
- **Phase 1**: 30 minutes - Basic HTML/CSS setup
- **Phase 2**: 45 minutes - Core JavaScript implementation  
- **Phase 3**: 60 minutes - Enhanced features (optional)
- **Phase 4**: 15 minutes - Testing and deployment
- **Total**: 2.5 hours for full implementation

## Next Steps
After basic implementation, you can enhance with:
- 7-day forecast display
- Weather charts/graphs
- Multiple location support
- Dark/light theme toggle
- Weather alerts and notifications