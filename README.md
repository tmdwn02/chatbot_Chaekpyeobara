# 💬 Chatbot template

A simple Streamlit app that shows how to build a chatbot using OpenAI's GPT-3.5.

[![Open in Streamlit](https://static.streamlit.io/badges/streamlit_badge_black_white.svg)](https://chatbot-template.streamlit.app/)

### How to run it on your own machine

1. Install the requirements

   ```
   $ pip install -r requirements.txt
   ```

2. Run the app

   ```
   $ streamlit run streamlit_app.py
   ```
import requests
import openai
import json
import streamlit as st

# 알라딘 API 키 (본인의 키로 교체)
TTB_KEY = "ttbtmdwn021442001"

# OpenAI API 키 (본인의 키로 교체)
openai.api_key = "your-openai-api-key"

# 알라딘 API를 통해 책 정보 가져오기
def search_book(book_title):
    search_url = "http://www.aladin.co.kr/ttb/api/ItemSearch.aspx"
    params = {
        "ttbkey": TTB_KEY,
        "Query": book_title,
        "QueryType": "Title",
        "MaxResults": 1,
        "SearchTarget": "Book",
        "output": "js",
        "Version": "20131101"
    }

    response = requests.get(search_url, params=params)
    data = response.json()

    if "item" in data and len(data["item"]) > 0:
        book = data["item"][0]
        book_info = {
            "title": book.get("title", "정보 없음"),
            "author": book.get("author", "정보 없음"),
            "publisher": book.get("publisher", "정보 없음"),
            "price": book.get("priceStandard", "정보 없음"),
            "isbn": book.get("isbn13", None),
            "description": book.get("description", "줄거리 정보 없음")
        }
        return book_info
    else:
        return {"error": "책을 찾을 수 없습니다."}

# GPT와의 대화
def chat_with_gpt(book_title, user_feedback):
    prompt = f"사용자가 '{book_title}'에 대해 다음과 같은 감상을 남겼습니다:\n\n{user_feedback}\n\n이 책에 대한 감상에 대해 더 이야기해 주세요."

    try:
        # 새로운 방식의 ChatCompletion 호출
        response = openai.ChatCompletion.create(
            model="gpt-4-turbo",  # 또는 사용 가능한 최신 모델
            messages=[
                {"role": "system", "content": "당신은 책 전문가입니다."},
                {"role": "user", "content": prompt}
            ],
            max_tokens=1500,
            temperature=1.0  # 응답의 창의성 조절
        )
        gpt_response = response['choices'][0]['message']['content']
        return gpt_response
    except Exception as e:
        return f"ChatGPT 호출 중 오류 발생: {e}"

# Streamlit 인터페이스
def main():
    st.title("📚 책과 대화하기")

    # 책 제목 입력 받기
    book_title = st.text_input("책 제목을 입력하세요:")

    # 책에 대한 감상 입력 받기
    user_feedback = st.text_area("책에 대한 감상을 입력하세요:")

    if st.button("검색하고 대화 시작"):
        if not book_title or not user_feedback:
            st.error("책 제목과 감상을 모두 입력하세요.")
            return
        
        # 책 정보 가져오기
        book_info = search_book(book_title)

        if "error" in book_info:
            st.error(f"오류: {book_info['error']}")
        else:
            # 책 정보 표시
            st.subheader("📖 책 정보")
            st.write(f"**제목:** {book_info['title']}")
            st.write(f"**저자:** {book_info['author']}")
            st.write(f"**출판사:** {book_info['publisher']}")
            st.write(f"**가격:** {book_info['price']}원")
            st.write(f"**ISBN:** {book_info['isbn']}")
            st.write(f"**줄거리:** {book_info['description']}")

            # 책 정보를 JSON 파일로 저장
            with open("book_info.json", "w", encoding="utf-8") as file:
                json.dump(book_info, file, ensure_ascii=False, indent=4)
            st.success("📂 책 정보를 'book_info.json' 파일로 저장했습니다.")

            # GPT 응답 받기
            gpt_response = chat_with_gpt(book_title, user_feedback)
            st.subheader("🤖 ChatGPT의 대답:")
            st.write(gpt_response)

if __name__ == "__main__":
    main()
