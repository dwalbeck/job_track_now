

https://www.npmjs.com/package/react-audio-voice-recorder?activeTab=readme
https://www.npmjs.com/package/react-audio-visualize



* Interview requested

* Check if "company" is created -> GET /v1/company/<company_id>
    
    - need to create "company" record entry -> POST /v1/company

* Generate company culture report -> GET /v1/company/culture/<company_id>

* Convert HTML resume to Markdown

    - Run conversion from HTML to MD /v1/convert/file

* Generate interview questions -> POST /v1/interview/question

    - create new "interview" record

    - write question blocks
    
    - create question entry for each question in list
    
    - create audio file for each question
    
    - return question list
    
* Start Interview

    - Authorize microphone use
    
    *****************************************************************
    **start of loop sequence**
    
    - Get audio for question -> POST /v1/interview/question/audio
    
    - Play audio
    
    - Display question text
    
    - Start recording from microphone

        - 30 sec interval (25mb max) send partial audio file for transcribing -> POST /v1/interview/transcribe
        
        - concatenate transcription text to answer
        
    - Send answer for processing -> POST /v1/interview/answer
    
        - Pull question data
        
        - AI call to evaluate answer and determine possible follow-up question
        
        - Score the answer and return results
        
            - Follow-up question requested
            
                - Add question to the DB
                
                - Create audio file of question

            - Update answer scores to question record in DB
            
            - Send response

    **next question for loop**
    ****************************************************************
    
* End interview

    - turn off mic recording
    
    - make an overall assessment of interview -> GET /v1/interview/<interview_id>
    
        - generate summary report
        
        - make AI call to score and provide feedback on interview
        
        - respond with results
        
    - update the interview record
    
    - respond with interview data and outcome











