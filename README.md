import time
from collections import defaultdict
from math import exp

class TimeDecayTracker:
    def __init__(self, decay_factor=0.5):
        """
        decay_factor: Controls how quickly past activities lose importance (higher = slower decay)
        """
        self.decay_factor = decay_factor
        self.activity_log = defaultdict(list)
        self.current_time = time.time()  # Base time for decay calculations

    def record_activity(self, user_id, action_type, value=1.0):
        """Record an activity with a timestamp and optional value"""
        timestamp = time.time()
        self.activity_log[user_id].append({
            'timestamp': timestamp,
            'action_type': action_type,
            'value': value
        })

    def _compute_decay(self, timestamp):
        """Calculate exponential decay based on time difference"""
        time_diff = (timestamp - self.current_time) / 3600  # Convert to hours
        return exp(self.decay_factor * time_diff)

    def get_current_score(self, user_id, action_types=None):
        """Get the current engagement score for a user, optionally filtered by action types"""
        if user_id not in self.activity_log:
            return 0.0

        self.current_time = time.time()  # Update current time for calculation
        filtered_activities = [
            act for act in self.activity_log[user_id]
            if action_types is None or act['action_type'] in action_types
        ]

        if not filtered_activities:
            return 0.0

        # Sum of time-decayed activity values
        return sum(act['value'] * self._compute_decay(act['timestamp']) for act in filtered_activities)

    def get_top_users(self, n=5, action_types=None):
        """Return the top n users by engagement score"""
        scores = {
            user_id: self.get_current_score(user_id, action_types)
            for user_id in self.activity_log
        }
        return sorted(scores.items(), key=lambda x: -x[1])[:n]

    def reset(self):
        """Clear all activity logs"""
        self.activity_log.clear()

# Example usage
if __name__ == "__main__":
    tracker = TimeDecayTracker(decay_factor=0.2)  # Strong decay (older activities fade faster)

    # Simulate user activities over time
    tracker.record_activity("user1", "click", 2.0)
    time.sleep(1)  # Simulate time passing
    tracker.record_activity("user1", "comment", 5.0)
    tracker.record_activity("user2", "click", 3.0)
    time.sleep(2)  # More time passes
    tracker.record_activity("user2", "share", 8.0)

    # Get current scores
    print("User1 current score:", tracker.get_current_score("user1"))
    print("User2 current score:", tracker.get_current_score("user2"))
    
    # Get top users
    print("Top users:", tracker.get_top_users())
