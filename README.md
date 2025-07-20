# Codealpha-internship-project
This repository contains my project for internship in codealpha.
// Enhanced AI-powered chatbot with retrieval-based responses and commercial features
'use client';

import React, { useState, useRef, useEffect } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { ScrollArea } from '@/components/ui/scroll-area';
import { Badge } from '@/components/ui/badge';
import { Separator } from '@/components/ui/separator';
import { Bot, User, Send, MessageCircle, Zap, RefreshCw } from 'lucide-react';

interface Message {
  id: string;
  text: string;
  sender: 'user' | 'bot';
  timestamp: Date;
  confidence?: number;
}

interface ResponsePattern {
  keywords: string[];
  responses: string[];
  category: string;
  confidence: number;
}

// Predefined response patterns for commercial use
const responsePatterns: ResponsePattern[] = [
  {
    keywords: ['hello', 'hi', 'hey', 'greetings', 'start'],
    responses: [
      'Hello! Welcome to our AI assistant. How can I help you today?',
      'Hi there! I\'m here to assist you. What would you like to know?',
      'Greetings! I\'m your AI chatbot. How may I assist you?'
    ],
    category: 'greeting',
    confidence: 0.9
  },
  {
    keywords: ['product', 'service', 'offer', 'what do you sell', 'catalog'],
    responses: [
      'We offer a wide range of products and services. Would you like me to show you our catalog?',
      'Our products include various categories. What specific type of product are you interested in?',
      'We have an extensive service portfolio. Let me know what you\'re looking for!'
    ],
    category: 'product_inquiry',
    confidence: 0.85
  },
  {
    keywords: ['price', 'cost', 'pricing', 'how much', 'expensive', 'cheap'],
    responses: [
      'Our pricing varies by product and service. Could you specify what you\'re interested in?',
      'We offer competitive pricing. Which product would you like pricing information for?',
      'Pricing depends on your specific needs. Let me connect you with our sales team for accurate quotes.'
    ],
    category: 'pricing',
    confidence: 0.8
  },
  {
    keywords: ['support', 'help', 'problem', 'issue', 'bug', 'error'],
    responses: [
      'I\'m here to help! Could you describe the issue you\'re experiencing?',
      'Our support team is ready to assist. What problem can I help you solve?',
      'Let me help you with that. Please provide more details about the issue.'
    ],
    category: 'support',
    confidence: 0.9
  },
  {
    keywords: ['contact', 'phone', 'email', 'address', 'location'],
    responses: [
      'You can reach us at support@company.com or call us at (555) 123-4567.',
      'Our contact information: Email: info@company.com, Phone: (555) 123-4567',
      'Feel free to contact us via email at support@company.com or visit our office!’
],
    category: 'contact',
    confidence: 0.95
  },
  {
    keywords: ['hours', 'open', 'closed', 'schedule', 'availability'],
    responses: [
      'We\'re open Monday to Friday, 9 AM to 6 PM EST. Weekend support is available via email.',
      'Our business hours are 9:00 AM - 6:00 PM, Monday through Friday.',
      'You can reach us during business hours: Mon-Fri 9AM-6PM. Emergency support is available 24/7.'
    ],
    category: 'hours',
    confidence: 0.9
  },
  {
    keywords: ['demo', 'trial', 'free', 'test', 'try'],
    responses: [
      'We offer a 14-day free trial! Would you like me to set that up for you?',
      'Yes, we have a free demo available. I can schedule one for you right now.',
      'Our free trial includes full access to all features. Shall I get you started?'
    ],
    category: 'trial',
    confidence: 0.85
  },
  {
    keywords: ['thanks', 'thank you', 'appreciate', 'grateful'],
    responses: [
      'You\'re very welcome! Is there anything else I can help you with?',
      'Happy to help! Feel free to ask if you have any other questions.',
      'My pleasure! I\'m here whenever you need assistance.'
    ],
    category: 'gratitude',
    confidence: 0.95
  }
];

const defaultResponses = [
  'I understand you\'re asking about something specific. Could you rephrase your question?',
  'I\'m still learning! Could you provide more details or try asking differently?',
  'That\'s an interesting question. Let me connect you with a human agent who can better assist you.',
  'I want to give you the most accurate information. Could you clarify what you\'re looking for?'
];

export default function ChatbotApp() {
  const [messages, setMessages] = useState<Message[]>([
    {
      id: '1',
      text: 'Hello! I\'m your AI assistant. I can help you with product information, pricing, support, and more. How can I assist you today?',
      sender: 'bot',
      timestamp: new Date(),
      confidence: 1.0
    }
  ]);
  const [inputValue, setInputValue] = useState('');
  const [isTyping, setIsTyping] = useState(false);
  const [isLoading, setIsLoading] = useState(false);
  const messagesEndRef = useRef<HTMLDivElement>(null);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };

  useEffect(() => {
    scrollToBottom();
  }, [messages]);

  const findBestResponse = (userMessage: string): { response: string; confidence: number; category: string } => {
    const lowercaseMessage = userMessage.toLowerCase();
    let bestMatch = { response: '', confidence: 0, category: 'unknown' };

    for (const pattern of responsePatterns) {
      const matchCount = pattern.keywords.filter(keyword => 
        lowercaseMessage.includes(keyword.toLowerCase())
      ).length;
      
      if (matchCount > 0) {
        const confidence = (matchCount / pattern.keywords.length) * pattern.confidence;
        if (confidence > bestMatch.confidence) {
          const randomResponse = pattern.responses[Math.floor(Math.random() * pattern.responses.length)];
          bestMatch = {
            response: randomResponse,
            confidence: confidence,
            category: pattern.category
          };
        }
      }
    }

    if (bestMatch.confidence < 0.3) {
      return {
        response: defaultResponses[Math.floor(Math.random() * defaultResponses.length)],
        confidence: 0.1,
        category: 'fallback'
      };
    }

    return bestMatch;
  };

  const handleSendMessage = async () => {
    if (!inputValue.trim()) return;

    const userMessage: Message = {
      id: Date.now().toString(),
      text: inputValue,
      sender: 'user',
      timestamp: new Date()
    };

    setMessages(prev => [...prev, userMessage]);
    setInputValue('');
    setIsTyping(true);
    setIsLoading(true);

    // Simulate AI processing time
    await new Promise(resolve => setTimeout(resolve, 1000 + Math.random() * 1000));

    const { response, confidence, category } = findBestResponse(inputValue);

    const botMessage: Message = {
      id: (Date.now() + 1).toString(),
      text: response,
      sender: 'bot',
      timestamp: new Date(),
      confidence: confidence
    };

    setMessages(prev => [...prev, botMessage]);
    setIsTyping(false);
    setIsLoading(false);
  };

  const handleKeyPress = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      handleSendMessage();
    }
  };

  const clearChat = () => {
    setMessages([{
      id: '1',
      text: 'Hello! I\'m your AI assistant. I can help you with product information, pricing, support, and more. How can I assist you today?',
      sender: 'bot',
      timestamp: new Date(),
      confidence: 1.0
    }]);
  };

  const formatTime = (date: Date) => {
    return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
  };

  return (
    <div className="flex flex-col h-screen max-h-screen bg-gray-50">
      {/* Header */}
      <Card className="rounded-none border-x-0 border-t-0">
        <CardHeader className="pb-3">
          <CardTitle className="flex items-center gap-2 text-lg">
<MessageCircle className="w-5 h-5 text-blue-600" />
            AI Customer Support
            <Badge variant="secondary" className="ml-2">
              <Zap className="w-3 h-3 mr-1" />
              Live
            </Badge>
          </CardTitle>
        </CardHeader>
      </Card>

      {/* Chat Messages */}
      <div className="flex-1 overflow-hidden">
        <ScrollArea className="h-full px-4">
          <div className="py-4 space-y-4">
            {messages.map((message) => (
              <div
                key={message.id}
                className={`flex gap-3 ${
                  message.sender === 'user' ? 'justify-end' : 'justify-start'
                }`}
              >
                {message.sender === 'bot' && (
                  <div className="flex-shrink-0 w-8 h-8 bg-blue-600 rounded-full flex items-center justify-center">
                    <Bot className="w-4 h-4 text-white" />
                  </div>
                )}
                
                <div className={`max-w-[70%] ${message.sender === 'user' ? 'order-2' : ''}`}>
                  <div
                    className={`p-3 rounded-2xl ${
                      message.sender === 'user'
                        ? 'bg-blue-600 text-white'
                        : 'bg-white border shadow-sm'
                    }`}
                  >
                    <p className="text-sm leading-relaxed">{message.text}</p>
                  </div>
                  <div className="flex items-center gap-2 mt-1 px-1">
                    <span className="text-xs text-gray-500">
                      {formatTime(message.timestamp)}
                    </span>
                    {message.sender === 'bot' && message.confidence && (
                      <Badge variant="outline" className="text-xs">
                        {Math.round(message.confidence * 100)}% confident
                      </Badge>
                    )}
                  </div>
                </div>

                {message.sender === 'user' && (
                  <div className="flex-shrink-0 w-8 h-8 bg-gray-600 rounded-full flex items-center justify-center order-3">
                    <User className="w-4 h-4 text-white" />
                  </div>
                )}
              </div>
            ))}

            {isTyping && (
              <div className="flex gap-3 justify-start">
                <div className="flex-shrink-0 w-8 h-8 bg-blue-600 rounded-full flex items-center justify-center">
                  <Bot className="w-4 h-4 text-white" />
                </div>
                <div className="max-w-[70%]">
                  <div className="p-3 rounded-2xl bg-white border shadow-sm">
                    <div className="flex space-x-1">
                      <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce"></div>
                      <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style={{ animationDelay: '0.1s' }}></div>
                      <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style={{ animationDelay: '0.2s' }}></div>
                    </div>
                  </div>
                </div>
              </div>
            )}
            
            <div ref={messagesEndRef} />
          </div>
        </ScrollArea>
      </div>

      <Separator />

      {/* Input Area */}
{/* Input Area */}
      <Card className="rounded-none border-x-0 border-b-0">
        <CardContent className="p-4">
          <div className="flex gap-2">
            <div className="flex-1">
              <Input
                value={inputValue}
                onChange={(e) => setInputValue(e.target.value)}
                onKeyPress={handleKeyPress}
                placeholder="Type your message here..."
                disabled={isLoading}
                className="min-h-[44px]"
              />
            </div>
            <Button
              onClick={handleSendMessage}
              disabled={!inputValue.trim() || isLoading}
              size="icon"
              className="min-w-[44px] h-[44px]"
            >
              <Send className="w-4 h-4" />
            </Button>
            <Button
              onClick={clearChat}
              variant="outline"
              size="icon"
              className="min-w-[44px] h-[44px]"
            >
              <RefreshCw className="w-4 h-4" />
            </Button>
          </div>
          
          {/* Quick Actions */}
          <div className="flex flex-wrap gap-2 mt-3">
            <Button
              variant="outline"
              size="sm"
              onClick={() => setInputValue('What products do you offer?')}
              className="text-xs"
            >
              Products
            </Button>
            <Button
              variant="outline"
              size="sm"
              onClick={() => setInputValue('What are your pricing options?')}
              className="text-xs"
            >
              Pricing
            </Button>
            <Button
              variant="outline"
              size="sm"
              onClick={() => setInputValue('I need support')}
              className="text-xs"
            >
              Support
            </Button>
            <Button
              variant="outline"
              size="sm"
              onClick={() => setInputValue('Can I try a demo?')}
              className="text-xs"
            >
              Demo
            </Button>
          </div>
        </CardContent>
      </Card>
    </div>
  );
}

