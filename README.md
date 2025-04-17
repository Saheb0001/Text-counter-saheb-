import streamlit as st
import re
from collections import Counter
import pandas as pd
import matplotlib.pyplot as plt

# App Title
st.set_page_config(page_title="WhatsApp Message Counter", layout="centered")
st.title("WhatsApp Chat Message Counter")
st.markdown("Upload your exported WhatsApp chat `.txt` file to see who messaged the most!")

# File Upload
uploaded_file = st.file_uploader("Upload your WhatsApp chat .txt file", type=["txt"])

# Regex Pattern to Match WhatsApp Message Format (supports AM/PM and 24-hour format)
pattern = re.compile(r"^\d{1,2}/\d{1,2}/\d{2,4}, \d{1,2}:\d{2}(?:\s?[APMapm]{2})? - ")

# Sender Extraction Function
def extract_sender(line):
    if pattern.match(line):
        try:
            message = line.split(" - ", 1)[1]
            if ": " in message:
                sender = message.split(": ", 1)[0]
                return sender
        except IndexError:
            return None
    return None

if uploaded_file is not None:
    # Read and Process Chat File
    chat_data = uploaded_file.read().decode("utf-8").splitlines()
    senders = [s for s in (extract_sender(line) for line in chat_data) if s]

    # Count Messages
    counts = Counter(senders)

    if counts:
        # Display Message Counts
        st.subheader("Message Count by Sender")
        for sender, count in counts.most_common():
            st.markdown(f"**{sender}**: {count} messages")

        # Bar Chart
        st.subheader("Bar Chart")
        st.bar_chart(pd.DataFrame(counts.items(), columns=["Sender", "Messages"]).set_index("Sender"))

        # Pie Chart
        st.subheader("Pie Chart")
        fig, ax = plt.subplots()
        ax.pie(counts.values(), labels=counts.keys(), autopct='%1.1f%%', startangle=90)
        ax.axis('equal')
        st.pyplot(fig)
    else:
        st.warning("No valid messages found in the uploaded file.")
else:
    st.info("Please upload a WhatsApp chat `.txt` file to begin.")
