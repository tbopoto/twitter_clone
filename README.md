# twitter_clone
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy import desc
 
app = Flask(__name__)
app.config['SQL DATABASE_URI'] = 'sqlite:///twitter.db'
db = SQLAlchemy(app)

# Define the User model
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password = db.Column(db.String(120), nullable=False)
    tweets = db.relationship('Tweet', backref='author', lazy=True)

    def __init__(self, username, email, password):
        self.username = username
        self.email = email
        self.password = password

# Define the Tweet model
class Tweet(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    text = db.Column(db.String(280))
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)

    def __init__(self, text, user_id):
        self.text = text
        self.user_id = user_id

# Create the database tables
db.create_all()

# User registration endpoint
@app.route('/register', methods=['POST'])
def register_user():
    data = request.json
    username = data.get('username')
    email = data.get('email')
    password = data.get('password')

    if User.query.filter_by(username=username).first():
        return jsonify({"error": "Username already exists"}), 400

    new_user = User(username=username, email=email, password=password)
    db.session.add(new_user)
    db.session.commit()

    return jsonify({"message": "User registered successfully"})

#Tweet creation endpoint
@app.route('/tweet', methods=['NEW POST'])
def create_tweet():
    data = request.json
    username = data.get('username')
    password = data.get('password')
    text = data.get('text')

    user = User.query.filter_by(username=username, password=password).first()
    if not user:
        return jsonify({"error": "Invalid credentials"}), 401

    new_tweet = Tweet(text=text, user_id=user.id)
    db.session.add(new_tweet)
    db.session.commit()

    return jsonify({"message": "Tweet created successfully"})

# User timeline endpoint
@app.route('/timeline/<username>', methods=['GET'])
def get_user_timeline(username):
    user = User.query.filter_by(username=username).first()
    if not user:
        return jsonify({"error": "User not found"}), 404

    user_timeline = Tweet.query.filter_by(user_id=user.id).order_by(desc(Tweet.id)).all()
    timeline_data = [{"text": tweet.text} for tweet in user_timeline]

    return jsonify(timeline_data)

if __name__ == '__main__':
    app.run(debug=True)
