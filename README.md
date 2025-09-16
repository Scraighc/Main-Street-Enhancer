# Main-Street-Enhancer
Main Street Enhancer

cargo init

# Import necessary libraries (available in the environment: flask for API, matplotlib for viz, numpy for calculations)
from flask import Flask, jsonify
import numpy as np
import matplotlib.pyplot as plt
import io
import base64  # For encoding plot as base64 if needed in API

app = Flask(__name__)

# Sample data for Fairmount Pilot (based on 2025 revitalization grant and local stats)
# Metrics: funding_impact (0-100, based on $40K grant relative to town size), population_stability (0-100, ~2,700 pop with low decline),
# economic_activity (0-100, e.g., downtown focus), community_engagement (0-100, e.g., Main Street input)
fairmount_metrics = {
    'funding_impact': 85,  # High due to OCRA $40K grant for planning
    'population_stability': 70,  # Stable bedroom community for Marion
    'economic_activity': 65,  # Boost from revitalization roadmap
    'community_engagement': 80  # Strong resident involvement via Main Street Fairmount
}

# Weights for composite score (customizable; sum to 1.0)
weights = {
    'funding_impact': 0.3,
    'population_stability': 0.25,
    'economic_activity': 0.25,
    'community_engagement': 0.2
}

def calculate_revitalization_score(metrics):
    """Calculate composite score (0-100)"""
    score = sum(metrics[key] * weights[key] for key in metrics)
    return round(score, 2)

@app.route('/api/revitalization-score/fairmount-pilot', methods=['GET'])
def get_fairmount_score():
    """API endpoint to return the score in JSON"""
    score = calculate_revitalization_score(fairmount_metrics)
    response = {
        'town': 'Fairmount Pilot',
        'description': 'Grant County, IN Downtown Revitalization Initiative (2025 OCRA Grant)',
        'score': score,
        'metrics_breakdown': fairmount_metrics,
        'interpretation': 'Score of 75+ indicates strong pilot potential for economic growth.'
    }
    return jsonify(response)

def visualize_score(metrics, score):
    """Generate a simple bar plot of metrics and overall score"""
    keys = list(metrics.keys())
    values = list(metrics.values())
    values.append(score)  # Add overall score
    keys.append('Overall Score')
    
    fig, ax = plt.subplots()
    ax.bar(keys, values, color=['blue', 'green', 'orange', 'red', 'purple'])
    ax.set_ylabel('Score (0-100)')
    ax.set_title('Fairmount Pilot Revitalization Score Breakdown')
    ax.set_ylim(0, 100)
    
    # Save plot to base64 for potential API embedding (or display locally)
    buf = io.BytesIO()
    plt.savefig(buf, format='png')
    buf.seek(0)
    plot_base64 = base64.b64encode(buf.read()).decode('utf-8')
    plt.close()
    return plot_base64  # Could return this in API for viz

# Example usage: Run the API and visualize
if __name__ == '__main__':
    # Calculate and print score
    score = calculate_revitalization_score(fairmount_metrics)
    print(f"Fairmount Pilot Revitalization Score: {score}/100")
    
    # Visualize (run this to see the plot)
    plot_data = visualize_score(fairmount_metrics, score)
    print("Plot generated (base64 encoded). In a full app, embed in HTML.")
    
    # Start API server (run with: python app.py, then visit http://localhost:5000/api/revitalization-score/fairmount-pilot)
    app.run(debug=True)
