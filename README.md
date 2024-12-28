# VotingInterface.js
To allow employees to vote and evaluate ideas through IMS-connect (Innovation Management System)

// frontend/src/components/VotingInterface.js

import React, { UseEffect, useState } from 'react';
import axios from 'axios';

const VotingInterface = () => {
    const [ideas, setIdeas] = UseState([]);
    const [error, setError] = UseState(null);

    UseEffect(() => {
        const fetchIdeas = async () => {
            try {
                const response = await Axios.get('/Api/ideas');
                setIdeas(response.data);
            } catch (err) {
                SetError('Failed to load ideas.');
            }
        };
        fetchIdeas();
    }, []);
    const handleVote = async (ideaId) => {
        try {
            await Axios.post(`/Api/vote`, { IdeaId });
            // Optionally refresh ideas or update local state to reflect the new vote
        } catch (err) {
            SetError('Failed to cast vote.');
        }
    };
  return (
        <div>
            <h2>Vote on Ideas</h2>
            {error && <p className="error">{error}</p>}
            <ul>
                {ideas.map((idea) => (
                    <li key={idea._id}>
                        <h3>{idea.title}</h3>
                        <p>{idea.description}</p>
                        <button onClick={() => handleVote(idea._id)}>Vote</button>
                        <span>Votes: {idea.votes}</span>
                    </li>
                ))}
            </ul>
        </div>
    );
};
export default VotingInterface;

// backend/src/models/Idea.js

const mongoose = require('mongoose');

const IdeaSchema = new mongoose.Schema({
    title: { type: String, required: true },
    description: { type: String, required: true },
    category: { type: String, required: false },
    SubmittedAt: { type: Date, default: Date.now },
    votes: { type: Number, default: 0 }, // New field for votes
});

module.exports = mongoose.model('Idea', IdeaSchema);

// backend/src/controllers/voteController.js

const Idea = require('../models/Idea');

exports.voteOnIdea = async (req, res) => {
    const { ideaId } = req.body;
    try {
        const idea = await Idea.findById(IdeaId);
        if (!idea) {
            return res.status(404).json({ error: 'Idea not found.' });
        }
        
        idea.votes += 1; // Increment the vote count
        await idea.save();
        res.status(200).json({ message: 'Vote cast successfully!' });
    } catch (error) {
        res.status(400).json({ error: 'Failed to cast vote.' });
    }
};

// backend/src/routes/voteRoutes.js

const express = require('express');
const router = express.Router();
const voteController = require('../controllers/voteController');

router.post('/', voteController.voteOnIdea);

module.exports = router;

// backend/src/server.js

const express = require('express');
const mongoose = require('mongoose');
const ideaRoutes = require('./routes/ideaRoutes');
const voteRoutes = require('./routes/voteRoutes');

const app = express();
app.use(express.json()); // for parsing application/json

// MongoDB connection
mongoose.connect('mongodb://localhost/green_future', {
    useNewUrlParser: true,
    useUnifiedTopology: true,
});

// Routes
app.use('/api/ideas', ideaRoutes);
app.use('/api/vote', voteRoutes); // Add voting routes

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
