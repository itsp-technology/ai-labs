# This is timer app code

![ScreenShot](/img/timer-app.png)

```
copy this url in the browser 

https://claude.ai/public/artifacts/1adb3a74-f7f9-40e2-95b8-5fa882b7d7af?fullscreen=true
```

```



import React, { useState, useEffect, useRef } from 'react';
import { Play, Pause, RotateCcw, Clock } from 'lucide-react';

export default function TimerApp() {
  const [time, setTime] = useState(0); // time in seconds
  const [inputHours, setInputHours] = useState(0);
  const [inputMinutes, setInputMinutes] = useState(5);
  const [inputSeconds, setInputSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  const [isFinished, setIsFinished] = useState(false);
  const [overtimeSeconds, setOvertimeSeconds] = useState(0);
  const [customNotifications, setCustomNotifications] = useState([15, 30]); // Default notifications in minutes
  const [newNotificationTime, setNewNotificationTime] = useState('');
  const [triggeredNotifications, setTriggeredNotifications] = useState(new Set());
  const intervalRef = useRef(null);
  const overtimeIntervalRef = useRef(null);
  const audioRef = useRef(null);

  useEffect(() => {
    if (isRunning && time > 0) {
      intervalRef.current = setInterval(() => {
        setTime(prevTime => {
          if (prevTime <= 1) {
            setIsRunning(false);
            setIsFinished(true);
            playAlarm();
            startOvertimeCounter();
            return 0;
          }
          return prevTime - 1;
        });
      }, 1000);
    } else {
      clearInterval(intervalRef.current);
    }

    return () => clearInterval(intervalRef.current);
  }, [isRunning, time]);

  // Overtime counter effect
  useEffect(() => {
    if (isFinished && overtimeSeconds >= 0) {
      overtimeIntervalRef.current = setInterval(() => {
        setOvertimeSeconds(prev => {
          const newSeconds = prev + 1;
          const newMinutes = Math.floor(newSeconds / 60);
          
          // Check for custom notifications
          customNotifications.forEach(notificationMinutes => {
            const notificationSeconds = notificationMinutes * 60;
            if (newSeconds === notificationSeconds && !triggeredNotifications.has(notificationMinutes)) {
              playNotificationSound();
              setTriggeredNotifications(prev => new Set([...prev, notificationMinutes]));
            }
          });
          
          return newSeconds;
        });
      }, 1000);
    } else {
      clearInterval(overtimeIntervalRef.current);
    }

    return () => clearInterval(overtimeIntervalRef.current);
  }, [isFinished, customNotifications, triggeredNotifications]);

  const playAlarm = () => {
    // Create a simple beep sound using Web Audio API
    const audioContext = new (window.AudioContext || window.webkitAudioContext)();
    const oscillator = audioContext.createOscillator();
    const gainNode = audioContext.createGain();
    
    oscillator.connect(gainNode);
    gainNode.connect(audioContext.destination);
    
    oscillator.frequency.value = 800;
    oscillator.type = 'sine';
    
    gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);
    gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 1);
    
    oscillator.start(audioContext.currentTime);
    oscillator.stop(audioContext.currentTime + 1);
  };

  const playNotificationSound = () => {
    // Play a series of beeps for notifications
    const audioContext = new (window.AudioContext || window.webkitAudioContext)();
    
    // Play 3 beeps
    for (let i = 0; i < 3; i++) {
      setTimeout(() => {
        const oscillator = audioContext.createOscillator();
        const gainNode = audioContext.createGain();
        
        oscillator.connect(gainNode);
        gainNode.connect(audioContext.destination);
        
        oscillator.frequency.value = 1000;
        oscillator.type = 'sine';
        
        gainNode.gain.setValueAtTime(0.2, audioContext.currentTime);
        gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.3);
        
        oscillator.start(audioContext.currentTime);
        oscillator.stop(audioContext.currentTime + 0.3);
      }, i * 400);
    }
  };

  const startOvertimeCounter = () => {
    setOvertimeSeconds(0);
    setTriggeredNotifications(new Set());
  };

  const addCustomNotification = () => {
    const minutes = parseInt(newNotificationTime);
    if (minutes && minutes > 0 && minutes <= 180 && !customNotifications.includes(minutes)) {
      setCustomNotifications(prev => [...prev, minutes].sort((a, b) => a - b));
      setNewNotificationTime('');
    }
  };

  const removeNotification = (minutesToRemove) => {
    setCustomNotifications(prev => prev.filter(min => min !== minutesToRemove));
    setTriggeredNotifications(prev => {
      const newSet = new Set(prev);
      newSet.delete(minutesToRemove);
      return newSet;
    });
  };

  const addQuickNotification = (minutes) => {
    if (!customNotifications.includes(minutes)) {
      setCustomNotifications(prev => [...prev, minutes].sort((a, b) => a - b));
    }
  };

  const startTimer = () => {
    if (time === 0) {
      const totalSeconds = inputHours * 3600 + inputMinutes * 60 + inputSeconds;
      if (totalSeconds > 0) {
        setTime(totalSeconds);
        setIsRunning(true);
        setIsFinished(false);
      }
    } else {
      setIsRunning(true);
      setIsFinished(false);
    }
  };

  const pauseTimer = () => {
    setIsRunning(false);
  };

  const resetTimer = () => {
    setIsRunning(false);
    setTime(0);
    setIsFinished(false);
    setOvertimeSeconds(0);
    setTriggeredNotifications(new Set());
    clearInterval(overtimeIntervalRef.current);
  };

  const formatTime = (seconds) => {
    const hours = Math.floor(seconds / 3600);
    const mins = Math.floor((seconds % 3600) / 60);
    const secs = seconds % 60;
    
    if (hours > 0) {
      return `${hours.toString().padStart(2, '0')}:${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    }
    return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
  };

  const getCircleProgress = () => {
    const totalTime = inputHours * 3600 + inputMinutes * 60 + inputSeconds;
    if (totalTime === 0) return 0;
    return ((totalTime - time) / totalTime) * 100;
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-indigo-900 via-purple-900 to-pink-900 flex items-center justify-center p-4">
      <div className="bg-white/10 backdrop-blur-lg rounded-3xl p-8 shadow-2xl border border-white/20 max-w-md w-full">
        <div className="text-center mb-8">
          <div className="flex items-center justify-center mb-4">
            <Clock className="w-8 h-8 text-white mr-2" />
            <h1 className="text-3xl font-bold text-white">Timer</h1>
          </div>
        </div>

        {/* Timer Display */}
        <div className="relative mb-8">
          <div className="w-64 h-64 mx-auto relative">
            {/* Background Circle */}
            <svg className="w-full h-full transform -rotate-90" viewBox="0 0 100 100">
              <circle
                cx="50"
                cy="50"
                r="45"
                fill="none"
                stroke="rgba(255,255,255,0.2)"
                strokeWidth="2"
              />
              {/* Progress Circle */}
              <circle
                cx="50"
                cy="50"
                r="45"
                fill="none"
                stroke="url(#gradient)"
                strokeWidth="3"
                strokeLinecap="round"
                strokeDasharray="283"
                strokeDashoffset={283 - (283 * getCircleProgress()) / 100}
                className="transition-all duration-1000 ease-linear"
              />
              <defs>
                <linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                  <stop offset="0%" stopColor="#06b6d4" />
                  <stop offset="100%" stopColor="#3b82f6" />
                </linearGradient>
              </defs>
            </svg>
            
            {/* Time Display */}
            <div className="absolute inset-0 flex items-center justify-center">
              <div className="text-center">
                <div className={`text-5xl font-mono font-bold ${isFinished ? 'text-red-400 animate-pulse' : 'text-white'}`}>
                  {isFinished ? `+${formatTime(overtimeSeconds)}` : formatTime(time)}
                </div>
                {isFinished && (
                  <div className="text-red-400 text-sm font-semibold mt-2 animate-bounce">
                    Time's Up! (+{Math.floor(overtimeSeconds / 60)}m {overtimeSeconds % 60}s)
                  </div>
                )}
              </div>
            </div>
          </div>
        </div>

        {/* Custom Notifications Setup */}
        {time === 0 && !isRunning && (
          <div className="mb-6">
            <div className="bg-white/10 border border-white/20 rounded-lg p-4 backdrop-blur-sm">
              <h3 className="text-white font-semibold mb-3 text-sm">Custom Notifications After Timer Ends</h3>
              
              {/* Current Notifications */}
              <div className="mb-3">
                <div className="flex flex-wrap gap-2 mb-2">
                  {customNotifications.map(minutes => (
                    <div key={minutes} className="flex items-center bg-blue-500/30 border border-blue-400/50 rounded-full px-3 py-1 text-xs text-white">
                      <span>{minutes}m</span>
                      <button
                        onClick={() => removeNotification(minutes)}
                        className="ml-2 text-white/70 hover:text-white text-lg leading-none"
                      >
                        ×
                      </button>
                    </div>
                  ))}
                </div>
              </div>

              {/* Add Custom Notification */}
              <div className="flex gap-2 mb-3">
                <input
                  type="number"
                  min="1"
                  max="180"
                  placeholder="Minutes"
                  value={newNotificationTime}
                  onChange={(e) => setNewNotificationTime(e.target.value)}
                  onKeyPress={(e) => e.key === 'Enter' && addCustomNotification()}
                  className="flex-1 h-8 px-2 text-sm bg-white/20 border border-white/30 rounded text-white placeholder-white/50 focus:outline-none focus:ring-1 focus:ring-blue-400"
                />
                <button
                  onClick={addCustomNotification}
                  className="bg-blue-500 hover:bg-blue-600 text-white px-3 py-1 rounded text-sm font-medium transition-colors"
                >
                  Add
                </button>
              </div>

              {/* Quick Add Buttons */}
              <div className="flex flex-wrap gap-1">
                <span className="text-white/70 text-xs mr-2">Quick add:</span>
                {[5, 10, 15, 20, 30, 45, 60, 90, 120].map(minutes => (
                  <button
                    key={minutes}
                    onClick={() => addQuickNotification(minutes)}
                    disabled={customNotifications.includes(minutes)}
                    className="bg-white/20 hover:bg-white/30 disabled:bg-white/10 disabled:text-white/40 text-white px-2 py-1 rounded text-xs transition-colors"
                  >
                    {minutes}m
                  </button>
                ))}
              </div>
            </div>
          </div>
        )}

        {/* Notification Status */}
        {isFinished && (
          <div className="mb-6">
            <div className="bg-red-500/20 border border-red-400/30 rounded-lg p-4 backdrop-blur-sm">
              <div className="text-red-200 text-sm font-semibold mb-3">Timer Completed - Overtime Tracking</div>
              <div className="grid grid-cols-2 gap-2 text-xs">
                {customNotifications.map(minutes => {
                  const isTriggered = triggeredNotifications.has(minutes);
                  const currentMinutes = Math.floor(overtimeSeconds / 60);
                  const isPending = currentMinutes < minutes;
                  
                  return (
                    <div key={minutes} className={`flex items-center gap-2 ${
                      isTriggered ? 'text-green-300' : isPending ? 'text-white/50' : 'text-yellow-300'
                    }`}>
                      <div className={`w-2 h-2 rounded-full ${
                        isTriggered ? 'bg-green-300' : isPending ? 'bg-white/30' : 'bg-yellow-300 animate-pulse'
                      }`}></div>
                      {minutes}m {isTriggered ? '✓ Notified' : isPending ? 'pending' : 'ready'}
                    </div>
                  );
                })}
              </div>
              {customNotifications.length === 0 && (
                <div className="text-white/50 text-xs italic">No notifications set</div>
              )}
            </div>
          </div>
        )}

        {/* Input Controls */}
        {time === 0 && !isRunning && (
          <div className="mb-6">
            <div className="flex items-center justify-center gap-3 mb-4">
              <div className="text-center">
                <label className="block text-white/70 text-sm mb-1">Hours</label>
                <input
                  type="number"
                  min="0"
                  max="23"
                  value={inputHours}
                  onChange={(e) => setInputHours(Math.max(0, Math.min(23, parseInt(e.target.value) || 0)))}
                  className="w-16 h-12 text-center text-lg font-bold bg-white/20 border border-white/30 rounded-lg text-white placeholder-white/50 focus:outline-none focus:ring-2 focus:ring-blue-400"
                />
              </div>
              <div className="text-white text-lg font-bold mt-6">:</div>
              <div className="text-center">
                <label className="block text-white/70 text-sm mb-1">Minutes</label>
                <input
                  type="number"
                  min="0"
                  max="59"
                  value={inputMinutes}
                  onChange={(e) => setInputMinutes(Math.max(0, Math.min(59, parseInt(e.target.value) || 0)))}
                  className="w-16 h-12 text-center text-lg font-bold bg-white/20 border border-white/30 rounded-lg text-white placeholder-white/50 focus:outline-none focus:ring-2 focus:ring-blue-400"
                />
              </div>
              <div className="text-white text-lg font-bold mt-6">:</div>
              <div className="text-center">
                <label className="block text-white/70 text-sm mb-1">Seconds</label>
                <input
                  type="number"
                  min="0"
                  max="59"
                  value={inputSeconds}
                  onChange={(e) => setInputSeconds(Math.max(0, Math.min(59, parseInt(e.target.value) || 0)))}
                  className="w-16 h-12 text-center text-lg font-bold bg-white/20 border border-white/30 rounded-lg text-white placeholder-white/50 focus:outline-none focus:ring-2 focus:ring-blue-400"
                />
              </div>
            </div>
          </div>
        )}

        {/* Control Buttons */}
        <div className="flex justify-center gap-4">
          {!isRunning ? (
            <button
              onClick={startTimer}
              disabled={time === 0 && inputMinutes === 0 && inputSeconds === 0}
              className="flex items-center gap-2 bg-green-500 hover:bg-green-600 disabled:bg-gray-500 disabled:cursor-not-allowed text-white px-6 py-3 rounded-full font-semibold transition-all duration-200 shadow-lg hover:shadow-xl hover:scale-105"
            >
              <Play className="w-5 h-5" />
              Start
            </button>
          ) : (
            <button
              onClick={pauseTimer}
              className="flex items-center gap-2 bg-yellow-500 hover:bg-yellow-600 text-white px-6 py-3 rounded-full font-semibold transition-all duration-200 shadow-lg hover:shadow-xl hover:scale-105"
            >
              <Pause className="w-5 h-5" />
              Pause
            </button>
          )}
          
          <button
            onClick={resetTimer}
            className="flex items-center gap-2 bg-red-500 hover:bg-red-600 text-white px-6 py-3 rounded-full font-semibold transition-all duration-200 shadow-lg hover:shadow-xl hover:scale-105"
          >
            <RotateCcw className="w-5 h-5" />
            Reset
          </button>
        </div>

        {/* Quick Preset Buttons */}
        {time === 0 && !isRunning && (
          <div className="mt-6">
            <div className="text-center text-white/70 text-sm mb-3">Quick presets:</div>
            <div className="flex justify-center gap-2 flex-wrap">
              {[
                { label: '30s', hours: 0, min: 0, sec: 30 },
                { label: '1m', hours: 0, min: 1, sec: 0 },
                { label: '3m', hours: 0, min: 3, sec: 0 },
                { label: '5m', hours: 0, min: 5, sec: 0 },
                { label: '10m', hours: 0, min: 10, sec: 0 },
                { label: '15m', hours: 0, min: 15, sec: 0 },
                { label: '30m', hours: 0, min: 30, sec: 0 },
                { label: '1h', hours: 1, min: 0, sec: 0 },
                { label: '2h', hours: 2, min: 0, sec: 0 },
                { label: '3h', hours: 3, min: 0, sec: 0 }
              ].map((preset) => (
                <button
                  key={preset.label}
                  onClick={() => {
                    setInputHours(preset.hours);
                    setInputMinutes(preset.min);
                    setInputSeconds(preset.sec);
                  }}
                  className="bg-white/20 hover:bg-white/30 text-white px-3 py-1 rounded-full text-sm transition-all duration-200 border border-white/30"
                >
                  {preset.label}
                </button>
              ))}
            </div>
          </div>
        )}
      </div>
    </div>
  );
}


```