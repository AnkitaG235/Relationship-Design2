# Relationship-Design2
Q 2-
server.js-
const express = require("express");
const mongoose = require("mongoose");
const dotenv = require("dotenv");

dotenv.config();
const app = express();
app.use(express.json());

// Connect to MongoDB
mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
}).then(() => console.log("Connected to MongoDB"))
  .catch(err => console.error("MongoDB Connection Error:", err));

// Import routes
const organizerRoutes = require("./routes/organizer.routes");
const eventRoutes = require("./routes/event.routes");
const attendeeRoutes = require("./routes/attendee.routes");
const registrationRoutes = require("./routes/registration.routes");

// Use routes
app.use("/organizers", organizerRoutes);
app.use("/events", eventRoutes);
app.use("/attendees", attendeeRoutes);
app.use("/registrations", registrationRoutes);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));

organizer.model.js-
const mongoose = require("mongoose");

const organizerSchema = new mongoose.Schema({
  name: { type: String, required: true },
  contact_info: String,
  events: [{ type: mongoose.Schema.Types.ObjectId, ref: "Event" }],
});

const Organizer = mongoose.model("Organizer", organizerSchema);
module.exports = Organizer;

event.model.js-
const mongoose = require("mongoose");

const eventSchema = new mongoose.Schema({
  title: { type: String, required: true, unique: true },
  date: { type: Date, required: true },
  location: String,
  organizer: { type: mongoose.Schema.Types.ObjectId, ref: "Organizer", required: true },
  attendees: [{ type: mongoose.Schema.Types.ObjectId, ref: "Registration" }],
});

const Event = mongoose.model("Event", eventSchema);
module.exports = Event;

attendee.model.js-
const mongoose = require("mongoose");

const attendeeSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  phone: String,
  registrations: [{ type: mongoose.Schema.Types.ObjectId, ref: "Registration" }],
});

const Attendee = mongoose.model("Attendee", attendeeSchema);
module.exports = Attendee;
registration.model.js-
const mongoose = require("mongoose");

const registrationSchema = new mongoose.Schema({
  event: { type: mongoose.Schema.Types.ObjectId, ref: "Event", required: true },
  attendee: { type: mongoose.Schema.Types.ObjectId, ref: "Attendee", required: true },
  registration_date: { type: Date, default: Date.now },
});

const Registration = mongoose.model("Registration", registrationSchema);
module.exports = Registration;
event.controller.js-
const Event = require("../models/event.model");

// Add a new event
const addEvent = async (req, res) => {
  try {
    const newEvent = new Event(req.body);
    await newEvent.save();
    res.status(201).json(newEvent);
  } catch (error) {
    res.status(500).json({ message: "Error adding event", error });
  }
};

// Get all events
const getAllEvents = async (req, res) => {
  try {
    const events = await Event.find().populate("organizer", "name");
    res.status(200).json(events);
  } catch (error) {
    res.status(500).json({ message: "Error fetching events", error });
  }
};

module.exports = { addEvent, getAllEvents };
event.routes.js-
const express = require("express");
const { addEvent, getAllEvents } = require("../controllers/event.controller");

const router = express.Router();

router.post("/", addEvent);
router.get("/", getAllEvents);

module.exports = router;
registration.controller.js-
const Registration = require("../models/registration.model");
const Event = require("../models/event.model");

const registerForEvent = async (req, res) => {
  const { eventId, attendeeId } = req.body;

  try {
    const existingRegistration = await Registration.findOne({ event: eventId, attendee: attendeeId });
    if (existingRegistration) {
      return res.status(400).json({ message: "Attendee is already registered for this event" });
    }

    const newRegistration = new Registration({ event: eventId, attendee: attendeeId });
    await newRegistration.save();

    res.status(201).json({ message: "Successfully registered", newRegistration });
  } catch (error) {
    res.status(500).json({ message: "Error registering for event", error });
  }
};

const cancelRegistration = async (req, res) => {
  const { registrationId } = req.params;

  try {
    const deletedRegistration = await Registration.findByIdAndDelete(registrationId);
    if (!deletedRegistration) return res.status(404).json({ message: "Registration not found" });

    res.status(200).json({ message: "Registration cancelled successfully" });
  } catch (error) {
    res.status(500).json({ message: "Error cancelling registration", error });
  }
};

module.exports = { registerForEvent, cancelRegistration };

registration.routes.js-const express = require("express");
const { registerForEvent, cancelRegistration } = require("../controllers/registration.controller");

const router = express.Router();

router.post("/register", registerForEvent);
router.delete("/:registrationId", cancelRegistration);

module.exports = router;
