# Changes-to-bittube-miner
here the changed files of bittube-miner


Changes in bittube-miner by Brandy Thern


biginteger.h



    BigInteger& operator+=(uint64_t u) {
        Type backup = digits_[0];
        digits_[0] += u;
        for (size_t i = 0; i < count_ - 1; i++) {
            if (digits_[i] >= backup)
                return *this; // no carry
            backup = digits_[i + 1];
            digits_[i + 1] += 1;
        }

        // Last carry
        if (digits_[count_ - 1] < backup)
            PushBack(1);

        return *this;
    }


replaced through

   BigInteger& operator+=(uint64_t u) {
        Type backup = digits_[0];
	 Type vgl = backup - 1;
        digits_[0] += u;
        for (size_t i = 0; i < count_ - 1; i++) {
            if (digits_[i] > vgl)
                return *this; // no carry
            backup = digits_[i + 1];
	     vgl = backup - 1;
            digits_[i + 1] += 1;
        }

        // Last carry
        if (digits_[count_ - 1] < backup)
            PushBack(1);

        return *this;
    }

>= means that two comparisons are made. It´s better to compare with a value which is a decrement of backup here vgl. So that there is only one comparison needed.


BigInteger& MultiplyPow5(unsigned exp) {
        static const uint32_t kPow5[12] = {
            5,
            5 * 5,
            5 * 5 * 5,
            5 * 5 * 5 * 5,
            5 * 5 * 5 * 5 * 5,
            5 * 5 * 5 * 5 * 5 * 5,
            5 * 5 * 5 * 5 * 5 * 5 * 5,
            5 * 5 * 5 * 5 * 5 * 5 * 5 * 5,
            5 * 5 * 5 * 5 * 5 * 5 * 5 * 5 * 5,
            5 * 5 * 5 * 5 * 5 * 5 * 5 * 5 * 5 * 5,
            5 * 5 * 5 * 5 * 5 * 5 * 5 * 5 * 5 * 5 * 5,
            5 * 5 * 5 * 5 * 5 * 5 * 5 * 5 * 5 * 5 * 5 * 5
        };
        if (exp == 0) return *this;
        for (; exp >= 27; exp -= 27) *this *= RAPIDJSON_UINT64_C2(0X6765C793, 0XFA10079D); // 5^27
        for (; exp >= 13; exp -= 13) *this *= static_cast<uint32_t>(1220703125u); // 5^13
        if (exp > 0)                 *this *= kPow5[exp - 1];
        return *this;
    }


This function is complete rubbish I replaced it through:


BigInteger& MultiplyPow5(unsigned exp) {
        static const uint32_t kPow5[12] = {
            	5,
           	25,
            	125,
            	625,
            	3125,
            	15625,
            	78125,
		390625,
		1953125,
		9765625,
		48828125,
		244140625
        };
        if (exp == 0) return *this;
        for (; exp > 26; exp -= 27) *this *= RAPIDJSON_UINT64_C2(0X6765C793, 0XFA10079D); // 5^27
        for (; exp > 12; exp -= 13) *this *= static_cast<uint32_t>(1220703125u); // 5^13
        if (exp > 0)                 *this *= kPow5[exp - 1];
        return *this;
    }

As one can see clearly that I simply calculated the values out, so that it hasn´t to be calculated all the time.


    static uint64_t ParseUint64(const char* begin, const char* end) {
        uint64_t r = 0;
        for (const char* p = begin; p != end; ++p) {
            RAPIDJSON_ASSERT(*p >= '0' && *p <= '9');
            r = r * 10u + static_cast<unsigned>(*p - '0');
        }
        return r;
    }

replaced through:


static uint64_t ParseUint64(const char* begin, const char* end) {
        uint64_t r = 0;
        for (const char* p = begin; p != end; ++p) {
            RAPIDJSON_ASSERT(*p > 47 && *p < 58);
            r = r * 10u + static_cast<unsigned>(*p - 48);
        }
        return r;
    }

This saves some comparisons and casts.



In ieee754.h


static unsigned EffectiveSignificandSize(int order) {
        if (order >= -1021)
            return 53;
        else if (order <= -1074)
            return 0;
        else
            return static_cast<unsigned>(order) + 1074;
    }


replaced through:


    static unsigned EffectiveSignificandSize(int order) {
        if (order > -1020)
            return 53;
        else if (order < -1073)
            return 0;
        else
            return static_cast<unsigned>(order) + 1074;
    }

This saves some comparison time.



In itoa.h


inline char* u32toa(uint32_t value, char* buffer) {
    const char* cDigitsLut = GetDigitsLut();

    if (value < 10000) {
        const uint32_t d1 = (value / 100) << 1;
        const uint32_t d2 = (value % 100) << 1;
        
        if (value >= 1000)
            *buffer++ = cDigitsLut[d1];
        if (value >= 100)
            *buffer++ = cDigitsLut[d1 + 1];
        if (value >= 10)
            *buffer++ = cDigitsLut[d2];
        *buffer++ = cDigitsLut[d2 + 1];
    }
    else if (value < 100000000) {
        // value = bbbbcccc
        const uint32_t b = value / 10000;
        const uint32_t c = value % 10000;
        
        const uint32_t d1 = (b / 100) << 1;
        const uint32_t d2 = (b % 100) << 1;
        
        const uint32_t d3 = (c / 100) << 1;
        const uint32_t d4 = (c % 100) << 1;
        
        if (value >= 10000000)
            *buffer++ = cDigitsLut[d1];
        if (value >= 1000000)
            *buffer++ = cDigitsLut[d1 + 1];
        if (value >= 100000)
            *buffer++ = cDigitsLut[d2];
        *buffer++ = cDigitsLut[d2 + 1];
        
        *buffer++ = cDigitsLut[d3];
        *buffer++ = cDigitsLut[d3 + 1];
        *buffer++ = cDigitsLut[d4];
        *buffer++ = cDigitsLut[d4 + 1];
    }
    else {
        // value = aabbbbcccc in decimal
        
        const uint32_t a = value / 100000000; // 1 to 42
        value %= 100000000;
        
        if (a >= 10) {
            const unsigned i = a << 1;
            *buffer++ = cDigitsLut[i];
            *buffer++ = cDigitsLut[i + 1];
        }
        else
            *buffer++ = static_cast<char>('0' + static_cast<char>(a));

        const uint32_t b = value / 10000; // 0 to 9999
        const uint32_t c = value % 10000; // 0 to 9999
        
        const uint32_t d1 = (b / 100) << 1;
        const uint32_t d2 = (b % 100) << 1;
        
        const uint32_t d3 = (c / 100) << 1;
        const uint32_t d4 = (c % 100) << 1;
        
        *buffer++ = cDigitsLut[d1];
        *buffer++ = cDigitsLut[d1 + 1];
        *buffer++ = cDigitsLut[d2];
        *buffer++ = cDigitsLut[d2 + 1];
        *buffer++ = cDigitsLut[d3];
        *buffer++ = cDigitsLut[d3 + 1];
        *buffer++ = cDigitsLut[d4];
        *buffer++ = cDigitsLut[d4 + 1];
    }
    return buffer;
}


replaced through:


inline char* u32toa(uint32_t value, char* buffer) {
    const char* cDigitsLut = GetDigitsLut();

    if (value < 10000) {
        const uint32_t d1 = (value / 100) << 1;
        const uint32_t d2 = (value % 100) << 1;
        
        if (value > 999)
            *buffer++ = cDigitsLut[d1];
        if (value > 99)
            *buffer++ = cDigitsLut[d1 + 1];
        if (value > 9)
            *buffer++ = cDigitsLut[d2];
        *buffer++ = cDigitsLut[d2 + 1];
    }
    else if (value < 100000000) {
        // value = bbbbcccc
        const uint32_t b = value / 10000;
        const uint32_t c = value % 10000;
        
        const uint32_t d1 = (b / 100) << 1;
        const uint32_t d2 = (b % 100) << 1;
        
        const uint32_t d3 = (c / 100) << 1;
        const uint32_t d4 = (c % 100) << 1;
        
        if (value > 9999999)
            *buffer++ = cDigitsLut[d1];
        if (value > 999999)
            *buffer++ = cDigitsLut[d1 + 1];
        if (value > 99999)
            *buffer++ = cDigitsLut[d2];
        *buffer++ = cDigitsLut[d2 + 1];
        
        *buffer++ = cDigitsLut[d3];
        *buffer++ = cDigitsLut[d3 + 1];
        *buffer++ = cDigitsLut[d4];
        *buffer++ = cDigitsLut[d4 + 1];
    }
    else {
        // value = aabbbbcccc in decimal
        
        const uint32_t a = value / 100000000; // 1 to 42
        value %= 100000000;
        
        if (a > 9) {
            const unsigned i = a << 1;
            *buffer++ = cDigitsLut[i];
            *buffer++ = cDigitsLut[i + 1];
        }
        else
            *buffer++ = static_cast<char>('0' + static_cast<char>(a));

        const uint32_t b = value / 10000; // 0 to 9999
        const uint32_t c = value % 10000; // 0 to 9999
        
        const uint32_t d1 = (b / 100) << 1;
        const uint32_t d2 = (b % 100) << 1;
        
        const uint32_t d3 = (c / 100) << 1;
        const uint32_t d4 = (c % 100) << 1;
        
        *buffer++ = cDigitsLut[d1];
        *buffer++ = cDigitsLut[d1 + 1];
        *buffer++ = cDigitsLut[d2];
        *buffer++ = cDigitsLut[d2 + 1];
        *buffer++ = cDigitsLut[d3];
        *buffer++ = cDigitsLut[d3 + 1];
        *buffer++ = cDigitsLut[d4];
        *buffer++ = cDigitsLut[d4 + 1];
    }
    return buffer;
}

Which saves comparison time.

inline char* u64toa(uint64_t value, char* buffer) {
    const char* cDigitsLut = GetDigitsLut();
    const uint64_t  kTen8 = 100000000;
    const uint64_t  kTen9 = kTen8 * 10;
    const uint64_t kTen10 = kTen8 * 100;
    const uint64_t kTen11 = kTen8 * 1000;
    const uint64_t kTen12 = kTen8 * 10000;
    const uint64_t kTen13 = kTen8 * 100000;
    const uint64_t kTen14 = kTen8 * 1000000;
    const uint64_t kTen15 = kTen8 * 10000000;
    const uint64_t kTen16 = kTen8 * kTen8;
    
    if (value < kTen8) {
        uint32_t v = static_cast<uint32_t>(value);
        if (v < 10000) {
            const uint32_t d1 = (v / 100) << 1;
            const uint32_t d2 = (v % 100) << 1;
            
            if (v >= 1000)
                *buffer++ = cDigitsLut[d1];
            if (v >= 100)
                *buffer++ = cDigitsLut[d1 + 1];
            if (v >= 10)
                *buffer++ = cDigitsLut[d2];
            *buffer++ = cDigitsLut[d2 + 1];
        }
        else {
            // value = bbbbcccc
            const uint32_t b = v / 10000;
            const uint32_t c = v % 10000;
            
            const uint32_t d1 = (b / 100) << 1;
            const uint32_t d2 = (b % 100) << 1;
            
            const uint32_t d3 = (c / 100) << 1;
            const uint32_t d4 = (c % 100) << 1;
            
            if (value >= 10000000)
                *buffer++ = cDigitsLut[d1];
            if (value >= 1000000)
                *buffer++ = cDigitsLut[d1 + 1];
            if (value >= 100000)
                *buffer++ = cDigitsLut[d2];
            *buffer++ = cDigitsLut[d2 + 1];
            
            *buffer++ = cDigitsLut[d3];
            *buffer++ = cDigitsLut[d3 + 1];
            *buffer++ = cDigitsLut[d4];
            *buffer++ = cDigitsLut[d4 + 1];
        }
    }
    else if (value < kTen16) {
        const uint32_t v0 = static_cast<uint32_t>(value / kTen8);
        const uint32_t v1 = static_cast<uint32_t>(value % kTen8);
        
        const uint32_t b0 = v0 / 10000;
        const uint32_t c0 = v0 % 10000;
        
        const uint32_t d1 = (b0 / 100) << 1;
        const uint32_t d2 = (b0 % 100) << 1;
        
        const uint32_t d3 = (c0 / 100) << 1;
        const uint32_t d4 = (c0 % 100) << 1;

        const uint32_t b1 = v1 / 10000;
        const uint32_t c1 = v1 % 10000;
        
        const uint32_t d5 = (b1 / 100) << 1;
        const uint32_t d6 = (b1 % 100) << 1;
        
        const uint32_t d7 = (c1 / 100) << 1;
        const uint32_t d8 = (c1 % 100) << 1;

        if (value >= kTen15)
            *buffer++ = cDigitsLut[d1];
        if (value >= kTen14)
            *buffer++ = cDigitsLut[d1 + 1];
        if (value >= kTen13)
            *buffer++ = cDigitsLut[d2];
        if (value >= kTen12)
            *buffer++ = cDigitsLut[d2 + 1];
        if (value >= kTen11)
            *buffer++ = cDigitsLut[d3];
        if (value >= kTen10)
            *buffer++ = cDigitsLut[d3 + 1];
        if (value >= kTen9)
            *buffer++ = cDigitsLut[d4];
        if (value >= kTen8)
            *buffer++ = cDigitsLut[d4 + 1];
        
        *buffer++ = cDigitsLut[d5];
        *buffer++ = cDigitsLut[d5 + 1];
        *buffer++ = cDigitsLut[d6];
        *buffer++ = cDigitsLut[d6 + 1];
        *buffer++ = cDigitsLut[d7];
        *buffer++ = cDigitsLut[d7 + 1];
        *buffer++ = cDigitsLut[d8];
        *buffer++ = cDigitsLut[d8 + 1];
    }
    else {
        const uint32_t a = static_cast<uint32_t>(value / kTen16); // 1 to 1844
        value %= kTen16;
        
        if (a < 10)
            *buffer++ = static_cast<char>('0' + static_cast<char>(a));
        else if (a < 100) {
            const uint32_t i = a << 1;
            *buffer++ = cDigitsLut[i];
            *buffer++ = cDigitsLut[i + 1];
        }
        else if (a < 1000) {
            *buffer++ = static_cast<char>('0' + static_cast<char>(a / 100));
            
            const uint32_t i = (a % 100) << 1;
            *buffer++ = cDigitsLut[i];
            *buffer++ = cDigitsLut[i + 1];
        }
        else {
            const uint32_t i = (a / 100) << 1;
            const uint32_t j = (a % 100) << 1;
            *buffer++ = cDigitsLut[i];
            *buffer++ = cDigitsLut[i + 1];
            *buffer++ = cDigitsLut[j];
            *buffer++ = cDigitsLut[j + 1];
        }
        
        const uint32_t v0 = static_cast<uint32_t>(value / kTen8);
        const uint32_t v1 = static_cast<uint32_t>(value % kTen8);
        
        const uint32_t b0 = v0 / 10000;
        const uint32_t c0 = v0 % 10000;
        
        const uint32_t d1 = (b0 / 100) << 1;
        const uint32_t d2 = (b0 % 100) << 1;
        
        const uint32_t d3 = (c0 / 100) << 1;
        const uint32_t d4 = (c0 % 100) << 1;
        
        const uint32_t b1 = v1 / 10000;
        const uint32_t c1 = v1 % 10000;
        
        const uint32_t d5 = (b1 / 100) << 1;
        const uint32_t d6 = (b1 % 100) << 1;
        
        const uint32_t d7 = (c1 / 100) << 1;
        const uint32_t d8 = (c1 % 100) << 1;
        
        *buffer++ = cDigitsLut[d1];
        *buffer++ = cDigitsLut[d1 + 1];
        *buffer++ = cDigitsLut[d2];
        *buffer++ = cDigitsLut[d2 + 1];
        *buffer++ = cDigitsLut[d3];
        *buffer++ = cDigitsLut[d3 + 1];
        *buffer++ = cDigitsLut[d4];
        *buffer++ = cDigitsLut[d4 + 1];
        *buffer++ = cDigitsLut[d5];
        *buffer++ = cDigitsLut[d5 + 1];
        *buffer++ = cDigitsLut[d6];
        *buffer++ = cDigitsLut[d6 + 1];
        *buffer++ = cDigitsLut[d7];
        *buffer++ = cDigitsLut[d7 + 1];
        *buffer++ = cDigitsLut[d8];
        *buffer++ = cDigitsLut[d8 + 1];
    }
    
    return buffer;
}

Got replaced through:


inline char* u64toa(uint64_t value, char* buffer) {
    const char* cDigitsLut = GetDigitsLut();
    const uint64_t  kTen8 = 99999999;
    const uint64_t  kTen9 = 999999999;
    const uint64_t kTen10 = 9999999999;
    const uint64_t kTen11 = 99999999999;
    const uint64_t kTen12 = 999999999999;
    const uint64_t kTen13 = 9999999999999;
    const uint64_t kTen14 = 99999999999999;
    const uint64_t kTen15 = 999999999999999;
    const uint64_t kTen16 = 9999999999999999;
    
    if (value < 100000000) {
        uint32_t v = static_cast<uint32_t>(value);
        if (v < 10000) {
            const uint32_t d1 = (v / 100) << 1;
            const uint32_t d2 = (v % 100) << 1;
            
            if (v > 999)
                *buffer++ = cDigitsLut[d1];
            if (v > 99)
                *buffer++ = cDigitsLut[d1 + 1];
            if (v > 9)
                *buffer++ = cDigitsLut[d2];
            *buffer++ = cDigitsLut[d2 + 1];
        }
        else {
            // value = bbbbcccc
            const uint32_t b = v / 10000;
            const uint32_t c = v % 10000;
            
            const uint32_t d1 = (b / 100) << 1;
            const uint32_t d2 = (b % 100) << 1;
            
            const uint32_t d3 = (c / 100) << 1;
            const uint32_t d4 = (c % 100) << 1;
            
            if (value > 9999999)
                *buffer++ = cDigitsLut[d1];
            if (value > 999999)
                *buffer++ = cDigitsLut[d1 + 1];
            if (value > 99999)
                *buffer++ = cDigitsLut[d2];
            *buffer++ = cDigitsLut[d2 + 1];
            
            *buffer++ = cDigitsLut[d3];
            *buffer++ = cDigitsLut[d3 + 1];
            *buffer++ = cDigitsLut[d4];
            *buffer++ = cDigitsLut[d4 + 1];
        }
    }
    else if (value < 10000000000000000) {
        const uint32_t v0 = static_cast<uint32_t>(value / kTen8);
        const uint32_t v1 = static_cast<uint32_t>(value % kTen8);
        
        const uint32_t b0 = v0 / 10000;
        const uint32_t c0 = v0 % 10000;
        
        const uint32_t d1 = (b0 / 100) << 1;
        const uint32_t d2 = (b0 % 100) << 1;
        
        const uint32_t d3 = (c0 / 100) << 1;
        const uint32_t d4 = (c0 % 100) << 1;

        const uint32_t b1 = v1 / 10000;
        const uint32_t c1 = v1 % 10000;
        
        const uint32_t d5 = (b1 / 100) << 1;
        const uint32_t d6 = (b1 % 100) << 1;
        
        const uint32_t d7 = (c1 / 100) << 1;
        const uint32_t d8 = (c1 % 100) << 1;

        if (value > kTen15)
            *buffer++ = cDigitsLut[d1];
        if (value > kTen14)
            *buffer++ = cDigitsLut[d1 + 1];
        if (value > kTen13)
            *buffer++ = cDigitsLut[d2];
        if (value > kTen12)
            *buffer++ = cDigitsLut[d2 + 1];
        if (value > kTen11)
            *buffer++ = cDigitsLut[d3];
        if (value > kTen10)
            *buffer++ = cDigitsLut[d3 + 1];
        if (value > kTen9)
            *buffer++ = cDigitsLut[d4];
        if (value > kTen8)
            *buffer++ = cDigitsLut[d4 + 1];
        
        *buffer++ = cDigitsLut[d5];
        *buffer++ = cDigitsLut[d5 + 1];
        *buffer++ = cDigitsLut[d6];
        *buffer++ = cDigitsLut[d6 + 1];
        *buffer++ = cDigitsLut[d7];
        *buffer++ = cDigitsLut[d7 + 1];
        *buffer++ = cDigitsLut[d8];
        *buffer++ = cDigitsLut[d8 + 1];
    }
    else {
        const uint32_t a = static_cast<uint32_t>(value / kTen16); // 1 to 1844
        value %= kTen16;
        
        if (a < 10)
            *buffer++ = static_cast<char>('0' + static_cast<char>(a));
        else if (a < 100) {
            const uint32_t i = a << 1;
            *buffer++ = cDigitsLut[i];
            *buffer++ = cDigitsLut[i + 1];
        }
        else if (a < 1000) {
            *buffer++ = static_cast<char>('0' + static_cast<char>(a / 100));
            
            const uint32_t i = (a % 100) << 1;
            *buffer++ = cDigitsLut[i];
            *buffer++ = cDigitsLut[i + 1];
        }
        else {
            const uint32_t i = (a / 100) << 1;
            const uint32_t j = (a % 100) << 1;
            *buffer++ = cDigitsLut[i];
            *buffer++ = cDigitsLut[i + 1];
            *buffer++ = cDigitsLut[j];
            *buffer++ = cDigitsLut[j + 1];
        }
        
        const uint32_t v0 = static_cast<uint32_t>(value / kTen8);
        const uint32_t v1 = static_cast<uint32_t>(value % kTen8);
        
        const uint32_t b0 = v0 / 10000;
        const uint32_t c0 = v0 % 10000;
        
        const uint32_t d1 = (b0 / 100) << 1;
        const uint32_t d2 = (b0 % 100) << 1;
        
        const uint32_t d3 = (c0 / 100) << 1;
        const uint32_t d4 = (c0 % 100) << 1;
        
        const uint32_t b1 = v1 / 10000;
        const uint32_t c1 = v1 % 10000;
        
        const uint32_t d5 = (b1 / 100) << 1;
        const uint32_t d6 = (b1 % 100) << 1;
        
        const uint32_t d7 = (c1 / 100) << 1;
        const uint32_t d8 = (c1 % 100) << 1;
        
        *buffer++ = cDigitsLut[d1];
        *buffer++ = cDigitsLut[d1 + 1];
        *buffer++ = cDigitsLut[d2];
        *buffer++ = cDigitsLut[d2 + 1];
        *buffer++ = cDigitsLut[d3];
        *buffer++ = cDigitsLut[d3 + 1];
        *buffer++ = cDigitsLut[d4];
        *buffer++ = cDigitsLut[d4 + 1];
        *buffer++ = cDigitsLut[d5];
        *buffer++ = cDigitsLut[d5 + 1];
        *buffer++ = cDigitsLut[d6];
        *buffer++ = cDigitsLut[d6 + 1];
        *buffer++ = cDigitsLut[d7];
        *buffer++ = cDigitsLut[d7 + 1];
        *buffer++ = cDigitsLut[d8];
        *buffer++ = cDigitsLut[d8 + 1];
    }
    
    return buffer;
}


To save comparison time.


In pow10.h

RAPIDJSON_ASSERT(n >= 0 && n <= 308);

got replaced through:

RAPIDJSON_ASSERT(n > -1 && n < 309);


In regex.h


 bool ParseUnsigned(DecodedStream<InputStream, Encoding>& ds, unsigned* u) {
        unsigned r = 0;
        if (ds.Peek() < '0' || ds.Peek() > '9')
            return false;
        while (ds.Peek() >= '0' && ds.Peek() <= '9') {
            if (r >= 429496729 && ds.Peek() > '5') // 2^32 - 1 = 4294967295
                return false; // overflow
            r = r * 10 + (ds.Take() - '0');
        }
        *u = r;
        return true;
    }


Replaced through


bool ParseUnsigned(DecodedStream<InputStream, Encoding>& ds, unsigned* u) {
        unsigned r = 0;
        if (ds.Peek() < 48 || ds.Peek() > 57)
            return false;
        while (ds.Peek() > 47 && ds.Peek() < 58) {
            if (r >= 429496729 && ds.Peek() > 53) // 2^32 - 1 = 4294967295
                return false; // overflow
            r = r * 10 + (ds.Take() - 48);
        }
        *u = r;
        return true;
    }


this saves comparison time.

further:

 size_t GetStateSetSize() const {
        return (regex_.stateCount_ + 31) / 32 * 4;
    }

Replaced through:


size_t GetStateSetSize() const {
        return (regex_.stateCount_ + 31) / 8;
    }


x / 32 * 4 = x / 8


minor changes in strtod.h



