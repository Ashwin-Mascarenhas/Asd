// app.js

const express = require('express');
const { google } = require('googleapis');
const app = express();
const port = 3000;

// Add your client ID and client secret here
const clientId = 'YOUR_CLIENT_ID';
const clientSecret = 'YOUR_CLIENT_SECRET';
const redirectUri = 'http://localhost:3000/auth/google/callback';

// Configure Google OAuth2 client
const oAuth2Client = new google.auth.OAuth2(clientId, clientSecret, redirectUri);

// Generate the Google OAuth2 URL for authorization
const scopes = ['https://www.googleapis.com/auth/calendar'];
const authUrl = oAuth2Client.generateAuthUrl({
  access_type: 'offline',
  scope: scopes,
});

// Handle the Google OAuth2 callback
app.get('/auth/google/callback', async (req, res) => {
  const { code } = req.query;

  try {
    // Exchange the authorization code for access and refresh tokens
    const { tokens } = await oAuth2Client.getToken(code);

    // Set the credentials for the OAuth2 client
    oAuth2Client.setCredentials(tokens);

    // Redirect the user to the homepage or any other desired route
    res.redirect('/');
  } catch (error) {
    console.error('Error authenticating:', error);
    res.status(500).send('Error authenticating with Google');
  }
});

// Create a calendar event
app.post('/events', async (req, res) => {
  const { name, startDateTime, endDateTime, attendees, description } = req.body;

  try {
    // Create a new calendar event resource
    const event = {
      summary: name,
      start: {
        dateTime: startDateTime,
      },
      end: {
        dateTime: endDateTime,
      },
      attendees: attendees.map((attendee) => ({ email: attendee })),
      description: description,
    };

    // Create the event in the user's primary calendar
    const calendar = google.calendar({ version: 'v3', auth: oAuth2Client });
    const response = await calendar.events.insert({
      calendarId: 'primary', // Use 'primary' for the user's primary calendar
      resource: event,
    });

    console.log('Event created:', response.data);

    res.status(200).send('Event created successfully');
  } catch (error) {
    console.error('Error creating event:', error);
    res.status(500).send('Error creating event');
  }
});

// Start the server
app.listen(port, () => {
  console.log(`Server listening at http://localhost:${port}`);
});
```

Make sure to replace `'YOUR_CLIENT_ID'` and `'YOUR_CLIENT_SECRET'` with your actual client ID and client secret obtained from the Google Cloud Console.
