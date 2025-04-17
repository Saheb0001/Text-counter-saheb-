import streamlit as st
import re
from collections import Counter

st.title("WhatsApp Chat Message Counter")

uploaded_file = st.file_uploader("Upload your WhatsApp chat .txt file", type=["txt"])

if uploaded_file is not None:
    chat_data = uploaded_file.read().decode("utf-8").splitlines()

    pattern = re.compile(r"^\d{2}/\d{2}/\d{4}, \d{1,2}:\d{2}\u202f(?:am|pm|AM|PM) - ")

    def extract_sender(line):
        try:
            sender_part = line.split(" - ", 1)[1]
            if ": " in sender_part:
                return sender_part.split(": ", 1)[0]
        except IndexError:
            return None
        return None

    senders = [extract_sender(line) for line in chat_data if pattern.match(line)]
    counts = Counter(senders)

    st.subheader("Message Count by Sender")
    for sender, count in counts.most_common():
        st.write(f"**{sender}**: {count} messages")

    st.bar_chart({sender: count for sender, count in counts.items()})
