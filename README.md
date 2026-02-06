# VoiceVault
Unlock real-time voice AI with secure, contextual enterprise intelligence
Inspiration
The inspiration struck while working on enterprise call center automation at a tech firm in Chandigarh. We used Elasticsearch for RAG-based search but hit walls turning it into voice agents—conversations felt robotic due to STT-LLM-TTS latency spikes over 2-3 seconds per turn. Real user frustration in noisy environments (accents, interruptions) mirrored your problem statement: text frameworks like Elastic Agent Builder excel at data retrieval but falter on natural voice dynamics. I aimed to bridge this for production-scale voice AI, automating 70% of routine queries without handoffs.
What I Learned
Diving deep revealed voice AI's core math: end-to-end latency L=L_STT+L_LLM+L_TTS, where L_STT dominates at ∼500ms for streaming models like Whisper Turbo, but balloons with accents. Context management needs vector stores with decay: similarity score s=cos⁡(q ⃗,(h_i ) ⃗)⋅e^(-λt_i ), weighting recent history h_i. RAG shines for private data (Elastic's kNN search), but hallucinations drop 40% only with hybrid retrieval (BM25 + dense vectors). Key takeaway: voice demands <1s E2E latency and interruption detection via VAD (voice activity detection) thresholds.
How I Built the Project
I extended Elastic Agent Builder into "VoiceFlow Agent," a low-latency voice wrapper:
	Streaming Pipeline: Used Deepgram for real-time STT (100ms latency) and ElevenLabs Edge TTS, piping audio via WebSockets to bypass full transcription waits.
	Conversation Engine: Custom FSM (finite state machine) on Elasticsearch for turns: query → RAG retrieve → LLM (Llama 3.1 8B quantized) → partial TTS streaming. Interruptions trigger via WebRTC echo cancellation.
	Memory Layer: Elastic's semantic search with time-decayed embeddings; stored turns as docs with TTL.
	RAG Integration: Ingested enterprise data (PDFs, tickets) via Elastic Ingest Pipelines, securing with API keys and row-level access.
	Deployment: Dockerized on Kubernetes with GKE, scaling to 100 concurrent calls; fallback to human via Twilio handoff.
Prototype handled 85% query accuracy on internal benchmarks.
Challenges Faced
	Latency Wars: Initial E2E hit 4s; optimized by speculative decoding in LLM (P(y|x)≈P_draft (y|x)), shaving 1.5s.
	Noisy Real-World Input: Accents dropped STT accuracy to 60%; fine-tuned Whisper on 10k hours of Hindi-English calls.
	Context Drift: Agent repeated queries 25% of time; fixed with exponential decay λ=0.1 per minute.
	Security Hurdles: Fragmented data risked leaks; implemented zero-trust with Elastic's SAML and audit logs.
	Edge Cases: Hallucinations (e.g., fabricating ticket IDs) needed guardrails like confidence thresholding p>0.9.
Overcame via iterative A/B tests on 1k simulated calls, boosting MOS score from 2.8 to 4.2.
