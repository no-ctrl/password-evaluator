<template>
  <div class="container">
    <section class="evaluator">
      <header>
        <h1>Password Strength Analyzer</h1>
        <p class="subheader">Enter a password to check its strength and get tips on making it more secure.</p>
      </header>

      <!-- Password Input Section -->
      <form @submit.prevent>
        <div class="visually-hidden">
          <label for="username">Username</label>
          <input
              type="text"
              id="username"
              name="username"
              autocomplete="username"
              tabindex="-1"
          />
        </div>
        <div class="input-wrapper">
          <input
              v-model="password"
              :type="showPassword ? 'text' : 'password'"
              placeholder="Enter your password"
              @input="evaluatePassword"
              class="password-input"
              autocomplete="new-password"
          />
          <button type="button" @click="showPassword = !showPassword" class="toggle-button">
            <svg v-if="!showPassword" viewBox="0 0 24 24" class="icon">
              <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/>
              <circle cx="12" cy="12" r="3"/>
            </svg>
            <svg v-else viewBox="0 0 24 24" class="icon">
              <path
                  d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"/>
              <line x1="1" y1="1" x2="23" y2="23"/>
            </svg>
          </button>
        </div>
      </form>

      <!-- Password Evaluation Results -->
      <div v-if="evaluation" class="results">
        <div class="score-container">
          <span :style="strengthLabelStyle" class="strength-label">{{ strengthLabel }}</span>
          <div class="score">
            <span>Score: </span>
            <span>{{ evaluation.score }}%</span>
          </div>
        </div>

        <div class="progress-bar">
          <div class="progress"
               :style="{ width: `${evaluation.score}%`, backgroundColor: evaluation.score < 30 ? '#e74c3c' : evaluation.score < 60 ? '#f39c12' : '#2ecc71' }"></div>
        </div>
        <div v-if="evaluation.commonPatterns.length" class="patterns">
          <h3 class="detected-patterns-title">Patterns:</h3>
          <p class="patterns-text">{{ evaluation.commonPatterns.join(', ') }}</p>
        </div>
        <!-- Evaluation Metrics Section -->
        <div v-if="evaluation.entropy || evaluation.breachCount" class="metrics">
          <!-- Entropy Metric -->
          <div v-if="evaluation.entropy" class="metric">
            <div class="metric-header">
              Entropy: <span class="metric-value">{{ evaluation.entropy.toFixed(1) }} bits</span>
            </div>
            <p>{{ getEntropyDescription(evaluation.entropy) }}</p>
          </div>

          <!-- Exposed in Breaches Metric -->
          <div v-if="evaluation.breachCount !== undefined" class="metric">
            <p>{{ getBreachDescription(evaluation.breachCount) }}</p>
            <div class="metric-header">
              Exposed in Breaches:
              <span :style="{ color: evaluation.breachCount === 0 ? '#2ecc71' : '#e74c3c', fontWeight: '600' }">{{
                  evaluation.breachCount
                }}</span>
            </div>
          </div>
        </div>

        <!-- Suggestions Section -->
        <div v-if="evaluation.feedback.length" class="suggestions">
          <h3 class="suggestions-title">Suggestions:</h3>
          <ul class="suggestions-list">
            <li v-for="(msg, idx) in evaluation.feedback" :key="idx">{{ msg }}</li>
          </ul>
        </div>
      </div>
    </section>
  </div>
</template>


<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', sans-serif;
  font-weight: 400;
  color: #333;
  background: #f7f9fc;
}

.container {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  padding: 1rem;
  background: linear-gradient(135deg, #7f5a83, #0d324d);
}

.visually-hidden {
  position: absolute;
  left: -10000px;
  top: auto;
  width: 1px;
  height: 1px;
  overflow: hidden;
}

.evaluator {
  width: 100%;
  max-width: 600px;
  background: white;
  border-radius: 16px;
  padding: 2rem 2.5rem;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
  text-align: center;
  transition: transform 0.3s ease;
}

.evaluator:hover {
  transform: translateY(-5px);
}

header h1 {
  font-size: 2rem;
  font-weight: 600;
  color: #2c3e50;
  margin-bottom: 0.5rem;
}

.subheader {
  font-size: 1rem;
  color: #7f8c8d;
  margin-bottom: 2rem;
}

.input-wrapper {
  position: relative;
  margin-bottom: 1.5rem;
}

.password-input {
  width: 100%;
  padding: 0.75rem 3rem 0.75rem 1rem;
  font-size: 1rem;
  border-radius: 8px;
  border: 1px solid #ccc;
  background-color: #f9f9f9;
  transition: all 0.3s ease;
}

.password-input:focus {
  border-color: #3498db;
  box-shadow: 0 0 5px rgba(52, 152, 219, 0.4);
  background-color: white;
}

.toggle-button {
  position: absolute;
  top: 50%;
  right: 1rem;
  transform: translateY(-50%);
  background: transparent;
  border: none;
  cursor: pointer;
}

.icon {
  width: 1.25rem;
  height: 1.25rem;
  color: #3498db;
  transition: color 0.3s;
}

.toggle-button:hover .icon {
  color: #555;
}

.progress-bar {
  height: 10px;
  background-color: #eee;
  border-radius: 5px;
  overflow: hidden;
  margin: 1.5rem 0;
}

.progress {
  height: 100%;
  transition: width 0.3s ease;
}

.metrics {
  margin-top: 1.5rem;
  text-align: left;
}

.metric {
  font-size: 0.95rem;
  color: #555;
  margin-bottom: 0.8rem;
}

.suggestions h3 {
  font-weight: 600;
  color: #333;
  margin-bottom: 0.5rem;
}

.suggestions ul {
  padding: 0;
  list-style-type: none;
  color: #666;
  font-size: 0.9rem;
}

.suggestions ul li {
  padding: 0.3rem 0;
  border-bottom: 1px solid #ddd;
}

.score-container {
  text-align: center;
}

.score {
  display: block;
  font-size: 1.5rem;
  font-weight: 600;
  margin-top: 0.5rem;
}

.strength-label {
  display: inline-block;
  font-size: 1.25rem;
  font-weight: bold;
  color: #333;
  margin-bottom: 0.3rem;
}

/* Mobile Responsiveness */
@media (max-width: 768px) {
  .evaluator {
    padding: 1.5rem;
  }

  header h1 {
    font-size: 1.5rem;
  }

  .subheader {
    font-size: 0.9rem;
    margin-bottom: 1.5rem;
  }

  .password-input {
    font-size: 0.9rem;
    padding: 0.5rem 2.5rem 0.5rem 0.75rem;
  }

  .icon {
    width: 1rem;
    height: 1rem;
  }

  .score-container {
    font-size: 1rem;
  }

  .strength-label {
    font-size: 1.1rem;
  }

  .patterns {
    margin-top: 1.5rem;
    font-family: Arial, sans-serif;
    text-align: left;
    padding: 0 1rem;
  }

  .detected-patterns-title {
    font-size: 1rem;
    font-weight: 600;
    color: #333;
    margin-bottom: 0.5rem;
    text-transform: uppercase;
    letter-spacing: 1px;
  }

  .patterns-text {
    font-size: 1rem;
    color: #555;
    margin: 0;
    line-height: 1.5;
    word-wrap: break-word;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .patterns-text span {
    margin-right: 0.5rem;
  }

  @media (max-width: 768px) {
    .patterns-text {
      font-size: 0.875rem;
    }

    .detected-patterns-title {
      font-size: 0.875rem;
    }
  }


}
</style>


<script>
import {defineComponent, ref, computed} from 'vue';
import CryptoJS from 'crypto-js';
import axios from 'axios';

export default defineComponent({
  name: 'EnhancedPasswordEvaluator',
  setup() {
    const password = ref('');
    const showPassword = ref(false);
    const evaluation = ref(null);
    const commonPasswords = ref(new Set());

    const BASE_LENGTH_SCORE = 2;
    const ENTROPY_MULTIPLIER = 0.5;
    const COMMON_PW_PENALTY = 50;
    const BREACH_MAX_PENALTY = 50;
    const MAX_SCORE = 100;

    const loadCommonPasswords = async () => {
      try {
        const cachedPasswords = localStorage.getItem('commonPasswords');
        if (cachedPasswords) {
          commonPasswords.value = new Set(cachedPasswords.split('\n'));
        } else {
          const response = await axios.get('/common_10000_passwords.txt');
          commonPasswords.value = new Set(response.data.split('\n'));
          localStorage.setItem('commonPasswords', response.data);
        }
      } catch (error) {
        console.error('Failed to load common passwords', error);
      }
    };

    loadCommonPasswords();

    const evaluatePassword = async () => {
      if (!password.value) {
        evaluation.value = null;
        return;
      }

      const feedback = [];
      let score = 0;


      const lengthScore = Math.min(password.value.length * BASE_LENGTH_SCORE, 20);
      score += lengthScore;

      if (password.value.length < 8) {
        feedback.push('Use at least 8 characters');
      } else if (password.value.length < 12) {
        feedback.push('Consider using more than 12 characters for stronger security');
      }


      const charTypes = {
        lowercase: /[a-z]/.test(password.value),
        uppercase: /[A-Z]/.test(password.value),
        number: /[0-9]/.test(password.value),
        special: /[^A-Za-z0-9]/.test(password.value),
      };

      Object.entries(charTypes).forEach(([type, present]) => {
        if (present) {
          const typeScore = Math.min(
              (password.value.match(new RegExp(`[${type === 'special' ? '^A-Za-z0-9' : type}]`, 'g')) || []).length * 5,
              20
          );
          score += typeScore;
        } else {
          feedback.push(`Add ${type} characters`);
        }
      });

      const isCommonPassword = commonPasswords.value.has(password.value.toLowerCase());
      if (isCommonPassword) {
        score -= COMMON_PW_PENALTY;
        feedback.push('This is a commonly used password');
      }


      const entropy = calculateEntropy(password.value);
      score += Math.min(entropy * ENTROPY_MULTIPLIER, 20);

      if (entropy < 40) {
        feedback.push('Entropy is low. Use more diverse characters.');
      }


      let breachCount = 0;
      try {
        const sha1 = CryptoJS.SHA1(password.value).toString();
        const prefix = sha1.substring(0, 5);
        const suffix = sha1.substring(5).toUpperCase();

        const response = await fetch(`https://api.pwnedpasswords.com/range/${prefix}`);
        const text = await response.text();

        const match = text.split('\n').find(line => line.startsWith(suffix));
        if (match) {
          breachCount = parseInt(match.split(':')[1]);
          score -= Math.min(BREACH_MAX_PENALTY, breachCount / 100);
          feedback.push(`Found in ${breachCount} data breaches`);
        }
      } catch (error) {
        console.error('Error checking breaches:', error);
      }

      // Нормализација на резултатот
      score = Math.max(0, Math.min(MAX_SCORE, score));

      evaluation.value = {
        score,
        entropy,
        breachCount,
        feedback,
        commonPatterns: analyzePatterns(password.value) || 'No obvious patterns detected',
      };
    };

    const calculateEntropy = (pwd) => {
      const charSetSize = new Set(
          pwd
              .split('')
              .map((char) => char.charCodeAt(0))
              .filter((code) => code < 65536) // Only valid UTF-16 characters
      ).size;

      return Math.log2(Math.pow(charSetSize, pwd.length));
    };

    const analyzePatterns = (pwd) => {
      const patterns = [];

      // Хелпер функции
      const hasRepeatedCharacters = (str) => /(.)(\1{2,})/.test(str);
      const isOnlyNumbers = (str) => /^[0-9]+$/.test(str);
      const isOnlyLetters = (str) => /^[a-zA-Z]+$/.test(str);
      const hasSequentialNumbers = (str) =>
          /(?:012|123|234|345|456|567|678|789|890)/.test(str);
      const hasSequentialLetters = (str) => {
        const alphabet = 'abcdefghijklmnopqrstuvwxyz';
        const lowerPwd = str.toLowerCase();
        for (let i = 0; i < lowerPwd.length - 2; i++) {
          if (alphabet.includes(lowerPwd.slice(i, i + 3))) return true;
        }
        return false;
      };
      const isPalindrome = (str) =>
          str.length > 3 &&
          str.toLowerCase() === str.toLowerCase().split('').reverse().join('');
      const hasAlternatingChars = (str) =>
          /^(?:[a-zA-Z][0-9]|[0-9][a-zA-Z])+$/.test(str);
      const hasSequentialAscii = (str) => {
        for (let i = 0; i < str.length - 2; i++) {
          if (
              str.charCodeAt(i) + 1 === str.charCodeAt(i + 1) &&
              str.charCodeAt(i + 1) + 1 === str.charCodeAt(i + 2)
          ) {
            return true;
          }
        }
        return false;
      };
      const hasRepeatedPattern = (str) => /^(.+)\1+$/.test(str);

      const containsKeyboardPattern = (str) => {
        const keyboardPatterns = ['qwerty', 'asdf', 'zxcv', '1234', '0987'];
        return keyboardPatterns.some((pattern) =>
            str.toLowerCase().includes(pattern)
        );
      };
      const hasRepeatedSpecialCharacters = (str) =>
          /([^a-zA-Z0-9])\1{2,}/.test(str);

      // Нови проверки
      const hasUppercaseOnly = (str) => /^[A-Z]+$/.test(str);
      const hasLowercaseOnly = (str) => /^[a-z]+$/.test(str);
      const hasComplexKeyboardPatterns = (str) => {
        const extendedPatterns = [
          'qwertyuiop',
          'asdfghjkl',
          'zxcvbnm',
          '0987654321',
          '1234567890',
        ];
        for (let i = 0; i < str.length - 3; i++) {
          const slice = str.slice(i, i + 4).toLowerCase();
          if (extendedPatterns.some((pattern) => pattern.includes(slice))) {
            return true;
          }
        }
        return false;
      };

      const isTooSimple = (str) =>
          str.length < 6 ||
          (isOnlyNumbers(str) && str.length < 8) ||
          (isOnlyLetters(str) && str.length < 8);
      const matchesCommonPatterns = (str) => {
        const regexPatterns = [
          /^(.)\1{5,}$/, // Six or more repeated characters
          /^[a-zA-Z]{3,}\d{2,}$/, // Letters followed by numbers
          /^\d{2,}[a-zA-Z]{3,}$/, // Numbers followed by letters
        ];
        return regexPatterns.some((regex) => regex.test(str));
      };


      if (hasRepeatedCharacters(pwd)) patterns.push('Repeated characters (e.g., aaa)');
      if (isOnlyNumbers(pwd)) patterns.push('Numbers only');
      if (isOnlyLetters(pwd)) patterns.push('Letters only');
      if (hasSequentialNumbers(pwd)) patterns.push('Consecutive numbers (e.g., 12345)');
      if (hasSequentialLetters(pwd)) patterns.push('Sequential letters (e.g., abc)');
      if (isPalindrome(pwd)) patterns.push('Palindrome (e.g., abba)');
      if (hasAlternatingChars(pwd)) patterns.push('Alternating letters and numbers (e.g., a1b2)');
      if (hasSequentialAscii(pwd)) patterns.push('Sequential ASCII characters (e.g., !@#)');
      if (hasRepeatedPattern(pwd)) patterns.push('Repeated patterns (e.g., abab)');
      if (containsKeyboardPattern(pwd)) patterns.push('Keyboard pattern (e.g., qwerty)');
      if (hasRepeatedSpecialCharacters(pwd)) patterns.push('Repeated special characters (e.g., $$$)');
      if (hasUppercaseOnly(pwd)) patterns.push('Uppercase letters only');
      if (hasLowercaseOnly(pwd)) patterns.push('Lowercase letters only');
      if (hasComplexKeyboardPatterns(pwd)) patterns.push('Complex keyboard patterns (e.g., qwertyuiop)');
      if (isTooSimple(pwd)) patterns.push('Too simple or too short');
      if (matchesCommonPatterns(pwd)) patterns.push('Matches common password patterns');


      if (patterns.length === 0) {
        patterns.push('No obvious patterns detected - likely secure');
      }

      return patterns;
    };


    const getEntropyDescription = (entropy) => {
      if (entropy >= 100) {
        return 'Exceptional entropy level - extremely strong against all types of attacks';
      }
      if (entropy >= 80) {
        return 'Very high entropy - excellent protection against brute force attacks';
      }
      if (entropy >= 60) {
        return 'Good entropy level - strong protection but could be improved';
      }
      if (entropy >= 40) {
        return 'Moderate entropy - vulnerable to advanced attack methods';
      }
      return 'Low entropy - highly vulnerable to common attack methods';
    };

    const getBreachDescription = (breachCount) => {
      if (breachCount > 1000000) return 'Critical: Immediate change recommended!';
      if (breachCount > 100000) return 'High Risk: Change strongly advised';
      if (breachCount > 0) return 'Warning: Consider changing';
      return 'Good news: Not found in any known data breaches';
    };

    const strengthLabel = computed(() => {
      const score = evaluation.value?.score || 0;
      if (score >= 90) return 'Very Strong';
      if (score >= 70) return 'Strong';
      if (score >= 50) return 'Moderate';
      if (score >= 30) return 'Weak';
      return 'Very Weak';
    });

    const strengthLabelStyle = computed(() => {
      const colorMap = {
        'Very Strong': '#4CAF50',
        Strong: '#8BC34A',
        Moderate: '#FFC107',
        Weak: '#FF9800',
        'Very Weak': '#F44336',
      };
      return {
        color: colorMap[strengthLabel.value] || '#000',
        fontWeight: 'bold',
        fontSize: '1.2em',
      };
    });

    return {
      password,
      showPassword,
      evaluation,
      evaluatePassword,
      strengthLabel,
      strengthLabelStyle,
      getEntropyDescription,
      getBreachDescription,
    };
  },
});
</script>

