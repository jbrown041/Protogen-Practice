Tech Brief: Open-Meteo Weather Widget Implementation
Overview
We'll implement a weather widget using Open-Meteo's free API on your existing HTML/CSS website hosted on Vercel. This solution requires no API keys, has no rate limits, and provides reliable weather data.
Technical Architecture
Data Flow
1. User visits page → Widget loads
2. Request geolocation → Browser prompts for permission
3. Fetch weather data → Call Open-Meteo API with coordinates
4. Display results → Update DOM with weather information
5. Handle errors → Graceful fallback for failed requests
API Endpoint
• Base URL: https://api.open-meteo.com/v1/forecast
• Method: GET
• Authentication: None required
• Rate Limits: None
Key Features
• Current weather conditions
• Temperature, humidity, wind speed
• Weather code (sunny, cloudy, rainy, etc.)
• Automatic location detection
• Fallback to manual location entry
• Responsive design
• Loading states and error handling
Technical Specifications
Browser Requirements
• Geolocation API support (95%+ browser coverage)
• Fetch API support (modern browsers)
• ES6+ JavaScript features
Performance Considerations
• Single API call on page load
• ~2KB additional JavaScript
• Minimal CSS overhead
• No external dependencies
Data Structure (Open-Meteo Response)
json Copy
{
  "current_weather": {
    "temperature": 72.5,
    "windspeed": 8.2,
    "winddirection": 240,
    "weathercode": 3,
    "time": "2026-03-18T15:00"
  },
  "current_weather_units": {
    "temperature": "°F",
    "windspeed": "mph"
  }
}

Step-by-Step Implementation Plan
Phase 1: Basic Setup (15 minutes)
Step 1: Create the HTML Structure
Add this to your existing HTML file where you want the widget:
html Copy
<!-- Weather Widget Container -->
<div id="weather-widget" class="weather-widget">
    <div class="weather-header">
        <h3>Current Weather</h3>
        <button id="refresh-weather" class="refresh-btn">↻</button>
    </div>
    <div id="weather-content" class="weather-content">
        <div class="weather-loading">
            <div class="loading-spinner"></div>
            <p>Loading weather data...</p>
        </div>
    </div>
    <div id="location-fallback" class="location-fallback" style="display: none;">
        <p>Unable to detect location</p>
        <input type="text" id="city-input" placeholder="Enter city name">
        <button id="search-weather">Get Weather</button>
    </div>
</div>
Step 2: Add CSS Styles
Create or add to your CSS file:
css Copy
/* Weather Widget Styles */
.weather-widget {
    background: linear-gradient(135deg, #74b9ff, #0984e3);
    color: white;
    padding: 20px;
    border-radius: 15px;
    max-width: 350px;
    margin: 20px auto;
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}
.weather-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px;
}
.weather-header h3 {
    margin: 0;
    font-size: 1.2em;
    font-weight: 600;
}
.refresh-btn {
    background: rgba(255, 255, 255, 0.2);
    border: none;
    color: white;
    padding: 8px 12px;
    border-radius: 50%;
    cursor: pointer;
    font-size: 16px;
    transition: background 0.3s ease;
}
.refresh-btn:hover {
    background: rgba(255, 255, 255, 0.3);
}
.weather-content {
    min-height: 120px;
    display: flex;
    flex-direction: column;
    justify-content: center;
}
.weather-loading {
    text-align: center;
}
.loading-spinner {
    width: 30px;
    height: 30px;
    border: 3px solid rgba(255, 255, 255, 0.3);
    border-top: 3px solid white;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin: 0 auto 10px;
}
@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}
.weather-info {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 15px;
    align-items: center;
}
.weather-main {
    text-align: center;
}
.temperature {
    font-size: 2.5em;
    font-weight: bold;
    margin: 0;
}
.weather-description {
    font-size: 0.9em;
    opacity: 0.9;
    margin: 5px 0;
}
.weather-details {
    font-size: 0.85em;
}
.weather-details div {
    margin: 5px 0;
    display: flex;
    justify-content: space-between;
}
.location-fallback {
    text-align: center;
    margin-top: 15px;
}
.location-fallback input {
    padding: 8px 12px;
    border: none;
    border-radius: 5px;
    margin: 10px 5px;
    width: 150px;
}
.location-fallback button {
    padding: 8px 15px;
    background: rgba(255, 255, 255, 0.2);
    border: none;
    color: white;
    border-radius: 5px;
    cursor: pointer;
}
.error-message {
    text-align: center;
    color: #ffcccb;
    font-size: 0.9em;
}
/* Responsive Design */
@media (max-width: 480px) {
    .weather-widget {
        margin: 10px;
        max-width: none;
    }
    
    .weather-info {
        grid-template-columns: 1fr;
        text-align: center;
    }
}
Phase 2: Core JavaScript Implementation (30 minutes)
Step 3: Create the Weather Service
Add this JavaScript to your HTML file or create a separate JS file:
javascript Copy
// Weather Widget Implementation
class WeatherWidget {
    constructor() {
        this.apiBase = 'https://api.open-meteo.com/v1';
        this.geocodingApi = 'https://geocoding-api.open-meteo.com/v1';
        this.init();
    }
init() {
        this.bindEvents();
        this.loadWeather();
    }
bindEvents() {
        document.getElementById('refresh-weather').addEventListener('click', () => {
            this.loadWeather();
        });
document.getElementById('search-weather').addEventListener('click', () => {
            this.searchByCity();
        });
document.getElementById('city-input').addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                this.searchByCity();
            }
        });
    }
async loadWeather() {
        this.showLoading();
        
        try {
            const position = await this.getCurrentPosition();
            const weatherData = await this.fetchWeatherData(
                position.coords.latitude, 
                position.coords.longitude
            );
            this.displayWeather(weatherData);
        } catch (error) {
            console.error('Weather loading error:', error);
            this.showLocationFallback();
        }
    }
getCurrentPosition() {
        return new Promise((resolve, reject) => {
            if (!navigator.geolocation) {
                reject(new Error('Geolocation not supported'));
                return;
            }
navigator.geolocation.getCurrentPosition(
                resolve,
                reject,
                { timeout: 10000, enableHighAccuracy: true }
            );
        });
    }
async fetchWeatherData(lat, lon) {
        const params = new URLSearchParams({
            latitude: lat,
            longitude: lon,
            current_weather: true,
            hourly: 'relativehumidity_2m',
            timezone: 'auto',
            temperature_unit: 'fahrenheit',
            windspeed_unit: 'mph'
        });
const response = await fetch(`${this.apiBase}/forecast?${params}`);
        
        if (!response.ok) {
            throw new Error(`Weather API error: ${response.status}`);
        }
return response.json();
    }
async searchByCity() {
        const cityName = document.getElementById('city-input').value.trim();
        if (!cityName) return;
this.showLoading();
try {
            // First, geocode the city name
            const geoResponse = await fetch(
                `${this.geocodingApi}/search?name=${encodeURIComponent(cityName)}&count=1`
            );
            const geoData = await geoResponse.json();
if (!geoData.results || geoData.results.length === 0) {
                throw new Error('City not found');
            }
const { latitude, longitude } = geoData.results[0];
            const weatherData = await this.fetchWeatherData(latitude, longitude);
            this.displayWeather(weatherData, cityName);
            
            // Hide fallback form
            document.getElementById('location-fallback').style.display = 'none';
        } catch (error) {
            this.showError('Unable to find weather for that city');
        }
    }
displayWeather(data, locationName = 'Your Location') {
        const weather = data.current_weather;
        const humidity = data.hourly?.relativehumidity_2m?.[0] || 'N/A';
        
        const weatherDescription = this.getWeatherDescription(weather.weathercode);
        const weatherIcon = this.getWeatherIcon(weather.weathercode);
const weatherHTML = `
            <div class="weather-info">
                <div class="weather-main">
                    <div class="weather-icon">${weatherIcon}</div>
                    <div class="temperature">${Math.round(weather.temperature)}°F</div>
                    <div class="weather-description">${weatherDescription}</div>
                    <div class="location">${locationName}</div>
                </div>
                <div class="weather-details">
                    <div><span>Wind:</span> <span>${weather.windspeed} mph</span></div>
                    <div><span>Direction:</span> <span>${this.getWindDirection(weather.winddirection)}</span></div>
                    <div><span>Humidity:</span> <span>${humidity}%</span></div>
                    <div><span>Updated:</span> <span>${this.formatTime(weather.time)}</span></div>
                </div>
            </div>
        `;
document.getElementById('weather-content').innerHTML = weatherHTML;
    }
showLoading() {
        document.getElementById('weather-content').innerHTML = `
            <div class="weather-loading">
                <div class="loading-spinner"></div>
                <p>Loading weather data...</p>
            </div>
        `;
        document.getElementById('location-fallback').style.display = 'none';
    }
showLocationFallback() {
        document.getElementById('weather-content').innerHTML = `
            <div class="error-message">
                <p>Unable to detect your location automatically</p>
            </div>
        `;
        document.getElementById('location-fallback').style.display = 'block';
    }
showError(message) {
        document.getElementById('weather-content').innerHTML = `
            <div class="error-message">
                <p>${message}</p>
            </div>
        `;
    }
getWeatherDescription(code) {
        const descriptions = {
            0: 'Clear sky',
            1: 'Mainly clear',
            2: 'Partly cloudy',
            3: 'Overcast',
            45: 'Foggy',
            48: 'Depositing rime fog',
            51: 'Light drizzle',
            53: 'Moderate drizzle',
            55: 'Dense drizzle',
            61: 'Slight rain',
            63: 'Moderate rain',
            65: 'Heavy rain',
            71: 'Slight snow',
            73: 'Moderate snow',
            75: 'Heavy snow',
            95: 'Thunderstorm',
            96: 'Thunderstorm with hail',
            99: 'Thunderstorm with heavy hail'
        };
        return descriptions[code] || 'Unknown';
    }
getWeatherIcon(code) {
        const icons = {
            0: '☀️', 1: '🌤️', 2: '⛅', 3: '☁️',
            45: '🌫️', 48: '🌫️',
            51: '🌦️', 53: '🌦️', 55: '🌦️',
            61: '🌧️', 63: '🌧️', 65: '⛈️',
            71: '🌨️', 73: '❄️', 75: '❄️',
            95: '⛈️', 96: '⛈️', 99: '⛈️'
        };
        return icons[code] || '🌡️';
    }
getWindDirection(degrees) {
        const directions = ['N', 'NE', 'E', 'SE', 'S', 'SW', 'W', 'NW'];
        return directions[Math.round(degrees / 45) % 8];
    }
formatTime(timeString) {
        return new Date(timeString).toLocaleTimeString([], { 
            hour: '2-digit', 
            minute: '2-digit' 
        });
    }
}
// Initialize the weather widget when the page loads
document.addEventListener('DOMContentLoaded', () => {
    new WeatherWidget();
});
Phase 3: Testing & Deployment (15 minutes)
Step 4: Test Locally
6. Open your HTML file in a browser
7. Allow location access when prompted
8. Verify weather data displays correctly
9. Test the refresh button
10. Test manual city search (deny location or use fallback)
Step 5: Deploy to Vercel
11. Commit your changes to your repository
12. Push to your connected branch
13. Vercel will automatically deploy
14. Test the live version
Phase 4: Enhancements (Optional)
Step 6: Additional Features
• Add 5-day forecast
• Include weather alerts
• Add temperature unit toggle (°F/°C)
• Implement caching to reduce API calls
• Add more detailed weather information
Troubleshooting Guide
Common Issues:
15. Geolocation denied: Fallback form should appear
16. API errors: Check browser console for details
17. CORS issues: Open-Meteo supports CORS, but check if you're testing locally
18. Styling issues: Verify CSS is properly linked
Browser Support:
• Chrome/Edge: Full support
• Firefox: Full support
• Safari: Full support
• Mobile browsers: Full support
This implementation provides a robust, free weather widget that will work reliably on your Vercel-hosted website!
