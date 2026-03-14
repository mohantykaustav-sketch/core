import { useState, useRef, useEffect } from 'react';

const SYSTEM_PROMPT = `You are ClarAI — a sharp, no-fluff decision-making coach built for young people facing real decisions. Your job is to cut through emotional noise and help users think clearly.

Your style:
- Direct, warm, a little edgy — like a smart older friend, not a therapist
- Short messages. Never write walls of text.
- Ask ONE powerful question at a time to uncover what they really want
- Use frameworks when helpful: pros/cons, 10/10/10 rule, regret minimization, values alignment
- End with a clear, honest take — don't be wishy-washy
- Never say "I'm just an AI" or be overly cautious
- Sign off as ClarAI, never mention Claude or Anthropic

Flow:
1. Ask what decision they're facing
2. Ask 2-3 clarifying questions (one at a time)
3. Reflect back what you're hearing
4. Give your honest take + recommended action

Keep responses under 80 words. Be memorable.`;

const suggestions = [
  'Should I drop my major?',
  'Stay or leave my relationship?',
  'Take the job offer or wait?',
  'Move cities after graduation?',
];

const FREE_LIMIT = 5;

export default function ClarAI() {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState('');
  const [loading, setLoading] = useState(false);
  const [started, setStarted] = useState(false);
  const [decisionCount, setDecisionCount] = useState(0);
  const [isIndia, setIsIndia] = useState(false);
  const [paywallStage, setPaywallStage] = useState(null);
  const [email, setEmail] = useState('');
  const [bonusDecisions, setBonusDecisions] = useState(0);
  const [pendingMessage, setPendingMessage] = useState(null);
  const bottomRef = useRef(null);

  useEffect(() => {
    const tz = Intl.DateTimeFormat().resolvedOptions().timeZone;
    setIsIndia(tz.startsWith('Asia/Kolkata') || tz.startsWith('Asia/Calcutta'));
  }, []);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages, loading, paywallStage]);

  const price = isIndia ? '₹50/month' : '$5/month';
  const launchPrice = isIndia ? '₹25' : '$2.50';
  const remaining = FREE_LIMIT + bonusDecisions - decisionCount;

  const sendMessage = async (text) => {
    const userText = text || input.trim();
    if (!userText || loading) return;

    if (decisionCount >= FREE_LIMIT + bonusDecisions && messages.length === 0) {
      setPendingMessage(userText);
      setPaywallStage('paywall');
      setInput('');
      return;
    }

    setInput('');
    setStarted(true);
    if (paywallStage) setPaywallStage(null);

    const newMessages = [...messages, { role: 'user', content: userText }];
    setMessages(newMessages);
    setLoading(true);
    if (messages.length === 0) setDecisionCount((c) => c + 1);

    try {
      const response = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'x-api-key':
            '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',
          'anthropic-version': '2023-06-01',
          'anthropic-dangerous-direct-browser-access': 'true',
        },
        body: JSON.stringify({
          model: 'claude-sonnet-4-20250514',
          max_tokens: 1000,
          system: SYSTEM_PROMPT,
          messages: newMessages,
        }),
      });
      const data = await response.json();
      const reply = data.content?.[0]?.text || 'Something went wrong.';
      setMessages([...newMessages, { role: 'assistant', content: reply }]);
    } catch {
      setMessages([
        ...newMessages,
        { role: 'assistant', content: 'Connection error. Try again.' },
      ]);
    }
    setLoading(false);
  };

  const handleEmailSubmit = (email) => {
    if (email.includes('@')) {
      setBonusDecisions((b) => b + 2);
      setPaywallStage('success');
    }
  };

  const handleContinueAfterEmail = () => {
    setPaywallStage(null);
    setMessages([]);
    if (pendingMessage) {
      setTimeout(() => sendMessage(pendingMessage), 100);
      setPendingMessage(null);
    }
  };

  const handlePaidContinue = () => {
    setPaywallStage(null);
    setDecisionCount(0);
    setBonusDecisions(999);
    setMessages([]);
    if (pendingMessage) {
      setTimeout(() => sendMessage(pendingMessage), 100);
      setPendingMessage(null);
    }
  };

  const dots = Array.from({ length: FREE_LIMIT }, (_, i) => i);

  return (
    <div
      style={{
        minHeight: '100vh',
        background: '#080808',
        fontFamily: "'Georgia', serif",
        display: 'flex',
        flexDirection: 'column',
        alignItems: 'center',
        justifyContent: 'center',
        padding: '20px',
        position: 'relative',
        overflow: 'hidden',
      }}
    >
      <div
        style={{
          position: 'fixed',
          inset: 0,
          opacity: 0.04,
          backgroundImage: `url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E")`,
          backgroundSize: '200px',
          pointerEvents: 'none',
        }}
      />
      <div
        style={{
          position: 'fixed',
          top: '-100px',
          left: '50%',
          transform: 'translateX(-50%)',
          width: '700px',
          height: '400px',
          background:
            'radial-gradient(ellipse, rgba(230,190,80,0.06) 0%, transparent 70%)',
          pointerEvents: 'none',
        }}
      />

      {paywallStage && (
        <div
          style={{
            position: 'fixed',
            inset: 0,
            zIndex: 100,
            background: 'rgba(0,0,0,0.9)',
            display: 'flex',
            alignItems: 'center',
            justifyContent: 'center',
            padding: '20px',
            backdropFilter: 'blur(12px)',
            animation: 'fadeIn 0.3s ease',
          }}
        >
          <div
            style={{
              background: '#0f0f0f',
              border: '1px solid rgba(230,190,80,0.25)',
              borderRadius: '8px',
              padding: '40px 36px',
              maxWidth: '420px',
              width: '100%',
              textAlign: 'center',
            }}
          >
            {paywallStage === 'paywall' && (
              <>
                <div
                  style={{
                    width: '48px',
                    height: '48px',
                    borderRadius: '50%',
                    background: 'linear-gradient(135deg, #e6be50, #c8982a)',
                    display: 'flex',
                    alignItems: 'center',
                    justifyContent: 'center',
                    margin: '0 auto 20px',
                    fontSize: '20px',
                    color: '#080808',
                    fontWeight: 'bold',
                    fontFamily: 'monospace',
                  }}
                >
                  C
                </div>
                <h2
                  style={{
                    color: '#f5f0e8',
                    fontSize: '22px',
                    margin: '0 0 8px',
                    fontStyle: 'italic',
                    fontWeight: 400,
                  }}
                >
                  You've reached your clarity limit
                </h2>
                <p
                  style={{
                    color: '#666',
                    fontSize: '13px',
                    fontFamily: 'monospace',
                    margin: '0 0 6px',
                    lineHeight: 1.7,
                  }}
                >
                  You've used all {FREE_LIMIT} free decisions.
                </p>
                {pendingMessage && (
                  <div
                    style={{
                      background: 'rgba(230,190,80,0.06)',
                      border: '1px solid rgba(230,190,80,0.15)',
                      borderRadius: '6px',
                      padding: '12px 16px',
                      marginBottom: '24px',
                      fontFamily: 'monospace',
                      color: '#888',
                      fontSize: '13px',
                      fontStyle: 'italic',
                      textAlign: 'left',
                    }}
                  >
                    "{pendingMessage}"
                  </div>
                )}
                <button
                  onClick={() => setPaywallStage('paid')}
                  style={{
                    width: '100%',
                    padding: '14px',
                    background: 'linear-gradient(135deg, #e6be50, #c8982a)',
                    border: 'none',
                    borderRadius: '4px',
                    color: '#080808',
                    fontSize: '13px',
                    fontFamily: 'monospace',
                    fontWeight: '700',
                    letterSpacing: '2px',
                    cursor: 'pointer',
                    marginBottom: '12px',
                  }}
                >
                  JOIN CLARAI PRO — {price}
                </button>
                <div
                  style={{
                    background: 'rgba(230,190,80,0.04)',
                    border: '1px solid rgba(230,190,80,0.1)',
                    borderRadius: '4px',
                    padding: '10px',
                    marginBottom: '16px',
                    fontFamily: 'monospace',
                    fontSize: '11px',
                    color: '#e6be50',
                  }}
                >
                  ⚡ LAUNCH OFFER — First month {launchPrice} only
                </div>
                <div
                  style={{
                    display: 'flex',
                    alignItems: 'center',
                    gap: '12px',
                    marginBottom: '16px',
                  }}
                >
                  <div
                    style={{
                      flex: 1,
                      height: '1px',
                      background: 'rgba(255,255,255,0.06)',
                    }}
                  />
                  <span
                    style={{
                      color: '#333',
                      fontSize: '11px',
                      fontFamily: 'monospace',
                    }}
                  >
                    or
                  </span>
                  <div
                    style={{
                      flex: 1,
                      height: '1px',
                      background: 'rgba(255,255,255,0.06)',
                    }}
                  />
                </div>
                <button
                  onClick={() => setPaywallStage('email')}
                  style={{
                    width: '100%',
                    padding: '13px',
                    background: 'transparent',
                    border: '1px solid rgba(255,255,255,0.08)',
                    borderRadius: '4px',
                    color: '#888',
                    fontSize: '12px',
                    fontFamily: 'monospace',
                    letterSpacing: '1.5px',
                    cursor: 'pointer',
                  }}
                >
                  GET 2 BONUS DECISIONS FREE →
                </button>
              </>
            )}

            {paywallStage === 'email' && (
              <EmailStage
                email={email}
                setEmail={setEmail}
                onSubmit={handleEmailSubmit}
                onBack={() => setPaywallStage('paywall')}
              />
            )}

            {paywallStage === 'success' && (
              <>
                <div
                  style={{
                    width: '48px',
                    height: '48px',
                    borderRadius: '50%',
                    border: '2px solid #e6be50',
                    display: 'flex',
                    alignItems: 'center',
                    justifyContent: 'center',
                    margin: '0 auto 20px',
                    fontSize: '20px',
                    color: '#e6be50',
                  }}
                >
                  ✓
                </div>
                <h2
                  style={{
                    color: '#e6be50',
                    fontSize: '20px',
                    margin: '0 0 8px',
                    fontStyle: 'italic',
                    fontWeight: 400,
                  }}
                >
                  You're on the list
                </h2>
                <p
                  style={{
                    color: '#666',
                    fontSize: '13px',
                    fontFamily: 'monospace',
                    margin: '0 0 28px',
                    lineHeight: 1.7,
                  }}
                >
                  2 bonus decisions unlocked. We'll hit you when Pro drops.
                </p>
                <button
                  onClick={handleContinueAfterEmail}
                  style={{
                    width: '100%',
                    padding: '14px',
                    background: 'linear-gradient(135deg, #e6be50, #c8982a)',
                    border: 'none',
                    borderRadius: '4px',
                    color: '#080808',
                    fontSize: '13px',
                    fontFamily: 'monospace',
                    fontWeight: '700',
                    letterSpacing: '2px',
                    cursor: 'pointer',
                  }}
                >
                  CONTINUE DECIDING →
                </button>
              </>
            )}

            {paywallStage === 'paid' && (
              <>
                <div
                  style={{
                    width: '48px',
                    height: '48px',
                    borderRadius: '50%',
                    background: 'linear-gradient(135deg, #e6be50, #c8982a)',
                    display: 'flex',
                    alignItems: 'center',
                    justifyContent: 'center',
                    margin: '0 auto 20px',
                    fontSize: '20px',
                  }}
                >
                  ⚡
                </div>
                <h2
                  style={{
                    color: '#f5f0e8',
                    fontSize: '20px',
                    margin: '0 0 8px',
                    fontStyle: 'italic',
                    fontWeight: 400,
                  }}
                >
                  Welcome to ClarAI Pro
                </h2>
                <p
                  style={{
                    color: '#666',
                    fontSize: '13px',
                    fontFamily: 'monospace',
                    margin: '0 0 28px',
                    lineHeight: 1.7,
                  }}
                >
                  Unlimited decisions. No limits. No noise.
                </p>
                <button
                  onClick={handlePaidContinue}
                  style={{
                    width: '100%',
                    padding: '14px',
                    background: 'linear-gradient(135deg, #e6be50, #c8982a)',
                    border: 'none',
                    borderRadius: '4px',
                    color: '#080808',
                    fontSize: '13px',
                    fontFamily: 'monospace',
                    fontWeight: '700',
                    letterSpacing: '2px',
                    cursor: 'pointer',
                  }}
                >
                  START DECIDING →
                </button>
              </>
            )}
          </div>
        </div>
      )}

      <div
        style={{
          width: '100%',
          maxWidth: '640px',
          display: 'flex',
          flexDirection: 'column',
        }}
      >
        <div style={{ textAlign: 'center', marginBottom: '32px' }}>
          <div
            style={{
              display: 'inline-flex',
              alignItems: 'center',
              gap: '8px',
              fontSize: '11px',
              letterSpacing: '4px',
              textTransform: 'uppercase',
              color: '#e6be50',
              marginBottom: '14px',
              fontFamily: 'monospace',
            }}
          >
            <span
              style={{
                width: '20px',
                height: '1px',
                background: '#e6be50',
                display: 'inline-block',
              }}
            />
            Decision Intelligence
            <span
              style={{
                width: '20px',
                height: '1px',
                background: '#e6be50',
                display: 'inline-block',
              }}
            />
          </div>
          <h1
            style={{
              fontSize: 'clamp(48px, 10vw, 76px)',
              fontWeight: '400',
              color: '#f5f0e8',
              margin: '0 0 6px',
              letterSpacing: '-2px',
              lineHeight: 1,
              fontStyle: 'italic',
            }}
          >
            ClarAI
          </h1>
          <p
            style={{
              color: '#555',
              fontSize: '12px',
              margin: 0,
              fontFamily: 'monospace',
              letterSpacing: '3px',
            }}
          >
            THINK SHARP. DECIDE FAST.
          </p>
          {bonusDecisions < 999 && (
            <div
              style={{
                marginTop: '16px',
                display: 'inline-flex',
                gap: '6px',
                alignItems: 'center',
              }}
            >
              {dots.map((i) => (
                <div
                  key={i}
                  style={{
                    width: '8px',
                    height: '8px',
                    borderRadius: '50%',
                    background: i < remaining ? '#e6be50' : '#1a1a1a',
                    border: `1px solid ${i < remaining ? '#e6be50' : '#333'}`,
                    transition: 'all 0.3s',
                  }}
                />
              ))}
              <span
                style={{
                  color: '#444',
                  fontSize: '11px',
                  fontFamily: 'monospace',
                  marginLeft: '6px',
                }}
              >
                {remaining} decision{remaining !== 1 ? 's' : ''} left
              </span>
            </div>
          )}
          {bonusDecisions >= 999 && (
            <div style={{ marginTop: '16px' }}>
              <span
                style={{
                  color: '#e6be50',
                  fontSize: '11px',
                  fontFamily: 'monospace',
                }}
              >
                ⚡ PRO — unlimited
              </span>
            </div>
          )}
        </div>

        <div
          style={{
            background: 'rgba(255,255,255,0.015)',
            border: '1px solid rgba(255,255,255,0.06)',
            borderRadius: '6px',
            minHeight: started ? '380px' : '0',
            maxHeight: '440px',
            overflowY: 'auto',
            padding: started ? '24px' : '0',
            display: 'flex',
            flexDirection: 'column',
            gap: '18px',
            transition: 'all 0.4s ease',
            scrollbarWidth: 'none',
          }}
        >
          {messages.map((m, i) => (
            <div
              key={i}
              style={{
                display: 'flex',
                justifyContent: m.role === 'user' ? 'flex-end' : 'flex-start',
                animation: 'fadeUp 0.3s ease',
              }}
            >
              {m.role === 'assistant' && (
                <div
                  style={{
                    width: '24px',
                    height: '24px',
                    borderRadius: '50%',
                    background: 'linear-gradient(135deg, #e6be50, #c8982a)',
                    display: 'flex',
                    alignItems: 'center',
                    justifyContent: 'center',
                    fontSize: '10px',
                    color: '#080808',
                    fontWeight: 'bold',
                    flexShrink: 0,
                    marginRight: '10px',
                    marginTop: '2px',
                    fontFamily: 'monospace',
                  }}
                >
                  C
                </div>
              )}
              <div
                style={{
                  maxWidth: '82%',
                  padding: '12px 16px',
                  borderRadius:
                    m.role === 'user'
                      ? '16px 16px 4px 16px'
                      : '4px 16px 16px 16px',
                  background:
                    m.role === 'user'
                      ? 'linear-gradient(135deg, #e6be50, #c8982a)'
                      : 'rgba(255,255,255,0.04)',
                  border:
                    m.role === 'assistant'
                      ? '1px solid rgba(255,255,255,0.07)'
                      : 'none',
                  color: m.role === 'user' ? '#080808' : '#ccc8c0',
                  lineHeight: '1.65',
                  fontFamily:
                    m.role === 'user' ? "'Georgia', serif" : 'monospace',
                  fontWeight: m.role === 'user' ? '600' : '400',
                  fontSize: m.role === 'assistant' ? '14px' : '15px',
                }}
              >
                {m.content}
              </div>
            </div>
          ))}
          {loading && (
            <div style={{ display: 'flex', gap: '6px', paddingLeft: '34px' }}>
              {[0, 1, 2].map((i) => (
                <div
                  key={i}
                  style={{
                    width: '6px',
                    height: '6px',
                    borderRadius: '50%',
                    background: '#e6be50',
                    animation: `pulse 1.2s ease-in-out ${i * 0.2}s infinite`,
                  }}
                />
              ))}
            </div>
          )}
          <div ref={bottomRef} />
        </div>

        {!started && (
          <div
            style={{
              display: 'flex',
              flexWrap: 'wrap',
              gap: '8px',
              margin: '16px 0',
            }}
          >
            {suggestions.map((s, i) => (
              <button
                key={i}
                onClick={() => sendMessage(s)}
                style={{
                  background: 'transparent',
                  border: '1px solid rgba(230,190,80,0.25)',
                  borderRadius: '2px',
                  color: '#888',
                  padding: '8px 14px',
                  fontSize: '12px',
                  fontFamily: 'monospace',
                  cursor: 'pointer',
                  transition: 'all 0.2s',
                }}
                onMouseEnter={(e) => {
                  e.target.style.color = '#e6be50';
                  e.target.style.borderColor = 'rgba(230,190,80,0.6)';
                }}
                onMouseLeave={(e) => {
                  e.target.style.color = '#888';
                  e.target.style.borderColor = 'rgba(230,190,80,0.25)';
                }}
              >
                {s}
              </button>
            ))}
          </div>
        )}

        <div
          style={{
            display: 'flex',
            border: '1px solid rgba(255,255,255,0.08)',
            borderRadius: '4px',
            overflow: 'hidden',
            marginTop: started ? '12px' : '0',
            background: 'rgba(255,255,255,0.02)',
          }}
        >
          <input
            value={input}
            onChange={(e) => setInput(e.target.value)}
            onKeyDown={(e) => e.key === 'Enter' && sendMessage()}
            placeholder={
              started ? 'keep going...' : 'what decision are you facing?'
            }
            style={{
              flex: 1,
              padding: '16px 20px',
              background: 'transparent',
              border: 'none',
              outline: 'none',
              color: '#f5f0e8',
              fontSize: '15px',
              fontFamily: "'Georgia', serif",
            }}
          />
          <button
            onClick={() => sendMessage()}
            disabled={loading || !input.trim()}
            style={{
              padding: '16px 24px',
              background: input.trim()
                ? 'linear-gradient(135deg, #e6be50, #c8982a)'
                : 'transparent',
              border: 'none',
              cursor: input.trim() ? 'pointer' : 'default',
              color: input.trim() ? '#080808' : '#333',
              fontSize: '20px',
              transition: 'all 0.2s',
              fontWeight: 'bold',
            }}
          >
            →
          </button>
        </div>

        <div
          style={{
            textAlign: 'center',
            marginTop: '20px',
            color: '#1e1e1e',
            fontSize: '10px',
            fontFamily: 'monospace',
            letterSpacing: '3px',
          }}
        >
          CLARAI © 2026 — DECISION INTELLIGENCE
        </div>
      </div>

      <style>{`
        @keyframes fadeUp {
          from { opacity: 0; transform: translateY(10px); }
          to { opacity: 1; transform: translateY(0); }
        }
        @keyframes fadeIn {
          from { opacity: 0; }
          to { opacity: 1; }
        }
        @keyframes pulse {
          0%, 100% { opacity: 0.2; transform: scale(0.8); }
          50% { opacity: 1; transform: scale(1.3); }
        }
        ::-webkit-scrollbar { display: none; }
        input::placeholder { color: #333; }
      `}</style>
    </div>
  );
}

function EmailStage({ email, setEmail, onSubmit, onBack }) {
  return (
    <>
      <div style={{ fontSize: '32px', marginBottom: '16px' }}>🎁</div>
      <h2
        style={{
          color: '#f5f0e8',
          fontSize: '20px',
          margin: '0 0 8px',
          fontStyle: 'italic',
          fontWeight: 400,
        }}
      >
        Get 2 bonus decisions free
      </h2>
      <p
        style={{
          color: '#666',
          fontSize: '13px',
          fontFamily: 'monospace',
          margin: '0 0 24px',
          lineHeight: 1.7,
        }}
      >
        Drop your email. We'll send you early access to ClarAI Pro when it
        launches.
      </p>
      <input
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        onKeyDown={(e) => e.key === 'Enter' && onSubmit(email)}
        placeholder="your@email.com"
        style={{
          width: '100%',
          padding: '13px 16px',
          background: 'rgba(255,255,255,0.03)',
          border: '1px solid rgba(255,255,255,0.08)',
          borderRadius: '4px',
          outline: 'none',
          color: '#f5f0e8',
          fontSize: '14px',
          fontFamily: 'monospace',
          boxSizing: 'border-box',
          marginBottom: '12px',
        }}
      />
      <button
        onClick={() => onSubmit(email)}
        style={{
          width: '100%',
          padding: '13px',
          background: email.includes('@')
            ? 'linear-gradient(135deg, #e6be50, #c8982a)'
            : 'rgba(255,255,255,0.04)',
          border: 'none',
          borderRadius: '4px',
          color: email.includes('@') ? '#080808' : '#444',
          fontSize: '13px',
          fontFamily: 'monospace',
          fontWeight: '700',
          letterSpacing: '2px',
          cursor: email.includes('@') ? 'pointer' : 'default',
          marginBottom: '12px',
        }}
      >
        UNLOCK 2 FREE DECISIONS →
      </button>
      <button
        onClick={onBack}
        style={{
          background: 'none',
          border: 'none',
          color: '#333',
          fontSize: '11px',
          fontFamily: 'monospace',
          cursor: 'pointer',
          letterSpacing: '1px',
        }}
      >
        ← back
      </button>
    </>
  );
}
