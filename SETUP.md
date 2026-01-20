# Developer Setup Guide
## Technical Documentation for Malay Language Adventure

This guide is for developers, contributors, and technical users who want to understand, modify, or deploy the Malay Language Adventure platform.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Technology Stack](#technology-stack)
4. [File Structure](#file-structure)
5. [Setup Instructions](#setup-instructions)
6. [Configuration](#configuration)
7. [API Integration](#api-integration)
8. [Database Schema](#database-schema)
9. [Development Workflow](#development-workflow)
10. [Deployment](#deployment)
11. [Contributing](#contributing)
12. [Troubleshooting](#troubleshooting)

---

## Project Overview

**Malay Language Adventure** is a single-page web application (SPA) built for Singapore primary school students to learn Malay language through AI-powered, adaptive quizzes.

### Key Capabilities

- User authentication and profile management
- AI-generated quiz questions using Google Gemini API
- Progress tracking with tier-based advancement system
- Daily vocabulary generation and caching
- Responsive design for all devices
- Knowledge base integration for educational content

---

## Architecture

### Application Type
**Single-Page Application (SPA)** - All functionality contained in one HTML file with embedded JavaScript.

### Architecture Pattern
**Client-Side Rendering** with:
- Frontend UI rendering
- API calls to external services
- Browser-based state management
- LocalStorage for caching

### Data Flow

```
User Action → JavaScript Event Handler → API Call (Gemini/Supabase)
                                              ↓
                                         Process Response
                                              ↓
                                      Update UI + LocalStorage
                                              ↓
                                    Persist to Supabase Database
```

---

## Technology Stack

### Frontend
- **HTML5** - Semantic markup structure
- **Tailwind CSS** (CDN) - Utility-first styling framework
- **JavaScript (ES6+)** - Modern JavaScript with modules
- **Google Fonts** - Inter & Baloo 2 font families

### Backend Services
- **Google Gemini 2.5 Flash API** - AI quiz and vocabulary generation
- **Supabase** - PostgreSQL database with REST API
- **Supabase Realtime** - Live data synchronization

### Storage
- **Browser LocalStorage** - Daily vocabulary caching
- **Supabase Database** - User profiles and progress persistence

### External Libraries
- **@supabase/supabase-js** (CDN via jsDelivr) - Supabase client library

---

## File Structure

```
johlls/
├── learnmalay.html              # Main application (SPA)
├── README.md                    # Project overview and welcome
├── LEARNING_GUIDE.md           # Comprehensive Malay language guide
├── USER_GUIDE.md               # User instructions for students/teachers/parents
├── SETUP.md                    # This file - developer documentation
├── SECURITY.md                 # Security considerations and best practices
├── Imbuhan_KB.md               # Knowledge base: Malay affixes
├── Peribahasa_KB.md            # Knowledge base: Proverbs and idioms
├── PenjodohBilangan_KB.md      # Knowledge base: Numerical classifiers
├── QuestionsBank_KB.md         # Knowledge base: Sample questions
└── Imbuhan Part 1.pdf          # Reference material PDF
```

### Main Application Structure (learnmalay.html)

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- Meta tags, title, fonts -->
    <!-- Tailwind CSS CDN -->
    <!-- Custom styles -->
  </head>
  <body>
    <!-- Header with user profile -->
    <!-- Main content area -->
      <!-- Login screen -->
      <!-- Dashboard with modules -->
      <!-- Quiz screen -->

    <script type="module">
      // Configuration (Supabase, API keys)
      // Knowledge bases (embedded)
      // User management functions
      // Quiz generation functions
      // Daily vocabulary functions
      // UI rendering functions
      // Event handlers
      // Initialization
    </script>
  </body>
</html>
```

---

## Setup Instructions

### Prerequisites

- **Web Browser** - Modern browser (Chrome, Firefox, Safari, Edge)
- **Text Editor** - VS Code, Sublime Text, or any code editor
- **Internet Connection** - Required for API calls and CDN resources
- **(Optional) Local Server** - For development testing (Live Server, Python SimpleHTTPServer, etc.)

### Quick Start

1. **Clone or Download Repository**
   ```bash
   git clone <repository-url>
   cd johlls
   ```

2. **Open in Browser**
   ```bash
   # Method 1: Double-click learnmalay.html

   # Method 2: Use a local server (recommended for development)
   # Using VS Code Live Server extension
   # Right-click learnmalay.html → "Open with Live Server"

   # Method 3: Python SimpleHTTPServer
   python3 -m http.server 8000
   # Then navigate to http://localhost:8000/learnmalay.html
   ```

3. **Test Functionality**
   - Register a new user
   - Try loading daily vocabulary
   - Take a quiz in one module
   - Check if score persists after refresh

---

## Configuration

### Current Configuration (In learnmalay.html)

The application currently has hardcoded configuration values in the `<script>` section:

```javascript
// Lines 69-71 (Supabase)
const SUPABASE_URL = 'https://qvowmphqmflfanjospqm.supabase.co';
const SUPABASE_KEY = 'eyJhbG...'; // Long JWT token
const supabase = createClient(SUPABASE_URL, SUPABASE_KEY);

// Line ~400 (Gemini API for quizzes)
const GEMINI_API_KEY = 'AIzaSyAn7vIqbxswzrW35CZC12SvEX3omjwwl2Q';

// Line ~556 (Gemini API for daily words)
const DAILY_WORDS_API_KEY = 'AIzaSyC9AKQcORYqndRZGhvTh-OqyMvE-Q7F2aI';
```

⚠️ **SECURITY WARNING:** These API keys are exposed in client-side code. See [SECURITY.md](./SECURITY.md) for mitigation strategies.

### Configuration Values Explained

#### Supabase Configuration

- **SUPABASE_URL:** Your Supabase project URL
- **SUPABASE_KEY:** Anon/public key for client-side access
  - This is the "anon" key, safe for client-side with proper RLS policies
  - However, consider security implications (see SECURITY.md)

#### Google Gemini API Keys

- **GEMINI_API_KEY:** Used for quiz question generation
- **DAILY_WORDS_API_KEY:** Used for daily vocabulary generation
  - Two separate keys allow for rate limit distribution
  - Can be the same key if preferred

### Knowledge Base Embedding

Knowledge bases are embedded directly in the JavaScript code (lines ~73-100):

```javascript
const KB = {
    imbuhan: { content: `... embedded markdown content ...` },
    peribahasa: { content: `... embedded markdown content ...` },
    tatabahasaPassages: { content: `... embedded markdown content ...` },
    penjodohBilangan: { content: `... embedded markdown content ...` }
};
```

**Alternative:** These could be loaded from external files via fetch() for better maintainability.

---

## API Integration

### Google Gemini API

#### Quiz Generation

**Endpoint:** `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent`

**Request Format:**
```javascript
{
  "contents": [{
    "parts": [{
      "text": "[System prompt + Knowledge base + Instructions]"
    }]
  }],
  "generationConfig": {
    "temperature": 0.9,
    "topK": 40,
    "topP": 0.95,
    "maxOutputTokens": 8192
  }
}
```

**Response Processing:**
- Extracts JSON from response text
- Parses quiz questions array
- Validates format (10 questions, MCQ structure)

#### Daily Vocabulary Generation

**Similar endpoint and structure** but different prompt:
- Generates 5 vocabulary words
- Includes Malay word, English meaning, example sentence
- Cached in LocalStorage for 24 hours

### Supabase Database

#### User Profile Operations

**Create User:**
```javascript
const { data, error } = await supabase
  .from('learners')
  .insert([{ username: 'Ali123', score: 0 }])
  .select();
```

**Fetch User:**
```javascript
const { data } = await supabase
  .from('learners')
  .from('learners')
  .eq('username', 'Ali123')
  .single();
```

**Update Score:**
```javascript
await supabase
  .from('learners')
  .update({ score: newScore })
  .eq('username', 'Ali123');
```

---

## Database Schema

### Supabase Table: `learners`

| Column | Type | Description | Constraints |
|--------|------|-------------|-------------|
| `id` | UUID | Primary key | Auto-generated |
| `username` | TEXT | User's chosen username | UNIQUE, NOT NULL |
| `score` | INTEGER | Accumulated points | DEFAULT 0 |
| `created_at` | TIMESTAMP | Account creation time | DEFAULT now() |

### Recommended Indexes

```sql
CREATE INDEX idx_learners_username ON learners(username);
CREATE INDEX idx_learners_score ON learners(score);
```

### Row Level Security (RLS)

**Current Setup:** Likely permissive or disabled for ease of use.

**Recommended Policies:**
```sql
-- Allow users to read only their own data
CREATE POLICY "Users can view own profile"
ON learners FOR SELECT
USING (auth.uid() = id);

-- Allow users to update only their own score
CREATE POLICY "Users can update own score"
ON learners FOR UPDATE
USING (auth.uid() = id);
```

⚠️ **Note:** Current implementation may not have proper RLS. Review SECURITY.md.

---

## Development Workflow

### Making Changes

1. **Edit learnmalay.html**
   - Use a code editor with syntax highlighting
   - Test frequently in browser

2. **Test Locally**
   - Open in browser or use local server
   - Test all user flows:
     - Registration
     - Login
     - Daily vocabulary loading
     - Quiz generation for all modules
     - Score persistence

3. **Validate**
   - Check browser console for errors
   - Test on multiple browsers
   - Test responsive design (mobile, tablet, desktop)

### Adding New Modules

To add a new learning module:

1. **Update Knowledge Base**
   ```javascript
   const KB = {
       // ... existing modules
       newModule: { content: `
       # New Module Content
       - Rules and examples
       ` }
   };
   ```

2. **Update Module Array**
   ```javascript
   const allModules = [
       { key: "imbuhan", title: "Imbuhan", tier: 0, description: "...", emoji: "🔤" },
       // ... existing modules
       { key: "newModule", title: "New Module", tier: 0, description: "...", emoji: "✨" }
   ];
   ```

3. **Test Quiz Generation**
   - Ensure AI can generate questions from new KB content

### Modifying AI Prompts

Quiz generation prompts are in the `generateQuiz()` function:

```javascript
const systemPrompt = `You are a Malay language tutor...`;
```

**Tips for prompt engineering:**
- Be specific about output format
- Provide clear examples
- Specify difficulty level based on tier
- Include error handling instructions

---

## Deployment

### Static Hosting Options

Since this is a single HTML file with no backend server needed:

#### Option 1: GitHub Pages

1. Push to GitHub repository
2. Go to Settings → Pages
3. Select branch and root folder
4. Access via `https://username.github.io/repo-name/learnmalay.html`

#### Option 2: Netlify

1. Drag and drop project folder to Netlify
2. Set build command: (none needed)
3. Set publish directory: `/`
4. Deploy

#### Option 3: Vercel

```bash
npm install -g vercel
vercel --prod
```

#### Option 4: Traditional Web Hosting

- Upload `learnmalay.html` and all `.md` files
- Ensure file permissions are correct
- Access via domain/learnmalay.html

### Environment Considerations

⚠️ **Before deploying:**

1. **Review API Keys** - Consider security implications (SECURITY.md)
2. **Test All Features** - Ensure APIs work in production
3. **Check CORS** - Gemini API and Supabase should allow your domain
4. **Monitor Quotas** - Watch API usage limits
5. **Set Up Analytics** (Optional) - Track usage patterns

---

## Contributing

### How to Contribute

We welcome contributions! You can help by:

1. **Fixing Bugs**
   - Check GitHub Issues for open bugs
   - Submit pull requests with fixes

2. **Adding Educational Content**
   - Enhance knowledge bases
   - Add more peribahasa or imbuhan examples
   - Improve AI prompts for better questions

3. **Improving UI/UX**
   - Enhance responsive design
   - Add accessibility features
   - Improve visual feedback

4. **Optimizing Performance**
   - Reduce API calls
   - Improve caching strategies
   - Optimize loading times

5. **Documentation**
   - Fix typos or unclear instructions
   - Add examples
   - Translate documentation

### Contribution Workflow

1. **Fork the repository**
2. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Make your changes**
4. **Test thoroughly**
5. **Commit with clear messages**
   ```bash
   git commit -m "Add: new peribahasa examples to knowledge base"
   ```
6. **Push to your fork**
7. **Submit a Pull Request**

### Code Style Guidelines

- **HTML:** Use semantic tags, proper indentation
- **CSS:** Follow Tailwind utility conventions
- **JavaScript:**
  - Use modern ES6+ syntax
  - Comment complex logic
  - Use descriptive variable names
  - Handle errors gracefully

---

## Troubleshooting

### Common Development Issues

#### Issue: API calls failing with CORS errors

**Solution:**
- Check browser console for exact error
- Verify API keys are correct
- Ensure Supabase project allows your origin
- Test with different browser (some block cross-origin)

#### Issue: Database operations not working

**Possible Causes:**
- Incorrect Supabase URL or key
- Table doesn't exist
- RLS policies blocking access

**Solutions:**
- Verify credentials in Supabase dashboard
- Check table exists with correct schema
- Review RLS policies
- Check browser network tab for error responses

#### Issue: Quiz questions not generating

**Possible Causes:**
- Gemini API key invalid or quota exceeded
- Malformed API request
- AI response doesn't match expected format

**Solutions:**
- Check API key in Google AI Studio
- Review quota limits in console
- Add error logging to see API response
- Verify JSON parsing logic

#### Issue: Daily vocabulary not updating

**Possible Causes:**
- LocalStorage caching issue
- Date comparison logic error
- API call failure

**Solutions:**
- Clear browser LocalStorage
- Check date format in code
- Verify second API key is valid
- Add console logs to debug flow

#### Issue: UI not responsive on mobile

**Solutions:**
- Test with browser dev tools device emulation
- Review Tailwind breakpoint classes (sm:, md:, lg:)
- Check viewport meta tag is present
- Verify no fixed widths breaking layout

### Debug Mode

Add debug logging by wrapping key functions:

```javascript
function debugLog(message, data) {
    if (window.location.hostname === 'localhost') {
        console.log(`[DEBUG] ${message}`, data);
    }
}

// Use throughout code
debugLog('User logged in', { username, score });
```

---

## Version History

### v2.16 (Current)
- Improved question variety
- Enhanced AI prompt engineering
- Optimized knowledge base integration
- Daily vocabulary system improvements

### Future Enhancements

**Potential features for future versions:**
- [ ] User authentication with passwords
- [ ] Teacher dashboard for student monitoring
- [ ] Detailed analytics and progress reports
- [ ] Spaced repetition algorithm
- [ ] Audio pronunciation for vocabulary
- [ ] Gamification badges and achievements
- [ ] Social features (study groups, leaderboards)
- [ ] Offline mode with service workers
- [ ] Mobile app version (React Native/Flutter)
- [ ] Multi-language support

---

## Resources

### Documentation Links

- [Supabase Documentation](https://supabase.com/docs)
- [Google Gemini API Docs](https://ai.google.dev/docs)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [MDN Web Docs](https://developer.mozilla.org/)

### Learning Resources

- [JavaScript ES6+ Features](https://es6-features.org/)
- [Async JavaScript](https://javascript.info/async)
- [Web APIs](https://developer.mozilla.org/en-US/docs/Web/API)

---

## Support

For technical questions or issues:

1. Check this documentation
2. Review [SECURITY.md](./SECURITY.md) for security-related questions
3. Search existing GitHub Issues
4. Open a new GitHub Issue with:
   - Clear description of problem
   - Steps to reproduce
   - Expected vs actual behavior
   - Browser and OS information
   - Console error messages

---

## License

Educational use encouraged. Please maintain attribution and contribute improvements back to the community.

---

*Happy coding! May your commits be bug-free and your merges conflict-free.* 🚀
