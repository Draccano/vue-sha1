<template>
  <div class="hello">
    <div class="p-20 flex items-center justify-center flex-col">
        <div class="my-4 text-4xl leading-tight xl:text-5xl font-semibold font-display">
          SHA-1 algorithm
        </div> 
       
        <div class="w-1/2 h-auto rounded max-w-xl shadow-lg py-4 bg-white flex flex-col">
          <div class="px-6 py-4 min-h-full">
            <label>
              <span class="text-sm font-semibold text-gray-700">Message</span>
              <textarea v-model="message" v-on:input="sha1" placeholder="Text" class="bg-gray-200 h-32 my-1 focus:bg-white appearance-none rounded w-full py-3 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" id="username"></textarea>
            </label>
          </div>
          <div class="px-1 block hidden">
            <button @click="sha1()">Calculate</button>
          </div>
          <div class="px-6 pb-4 pt-2" :class="{'hidden': !hash}">
            <span class="text-sm font-semibold text-gray-700 block">Hash</span>
            <div class="block bg-gray-200 my-1 rounded w-full py-2 px-3 text-gray-700 leading-tight flex justify-between">
              <div>
                {{ hash }}
              </div>
              <div>
                <button @click="clearHash" type="button" class="px-2 text-gray-500 hover:text-gray-600 focus:outline-none">
                  <svg class="h-5 w-5 fill-current" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 24 24" version="1.1">
                      <path fill-rule="evenodd" d="M13.59 12l7.066-7.067a1.123 1.123 0 10-1.589-1.589L12 10.411 4.933 3.344a1.123 1.123 0 10-1.59 1.59L10.412 12l-7.067 7.067a1.123 1.123 0 101.589 1.589L12 13.589l7.067 7.067a1.12 1.12 0 001.59 0c.438-.439.438-1.15 0-1.59L13.588 12z"/>
                  </svg>  
                </button>
              </div>
              
            </div>
            
          </div>
        </div>
      

      </div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  mounted() {
    
  },
  data() {
    return {
      message: '',
      h0: '',
      h1: '',
      h2: '',
      h3: '',
      h4: '',
      textAscii: '',
      byteArray: '',
      byteString: '',
      blocks: '',
      wordsOfBlocks: '',
      hash: '',
    }
  },
  methods: {
    sha1(){
      
       //Konstanty podle RFC 3174 standardu
      let h0 = this.padZero(this.hexToBinary('67452301'), 32); //01100111010001010010001100000001
      let h1 = this.padZero(this.hexToBinary('EFCDAB89'), 32); //11101111110011011010101110001001
      let h2 = this.padZero(this.hexToBinary('98BADCFE'), 32); //10011000101110101101110011111110
      let h3 = this.padZero(this.hexToBinary('10325476'), 32); //00010000001100100101010001110110
      let h4 = this.padZero(this.hexToBinary('C3D2E1F0'), 32); //00010000001100100101010001110110
      
      //Vytvoření pole Ascii textu z inputu
      this.textAscii = this.message.split('').map((letter) =>  letter.charCodeAt(0));

      //doplnění nul do 8 ciferného binárního čísla (prevedeno z dekadickeho)
      this.byteArray = this.textAscii
        .map((charNum) => charNum.toString(2))
        .map((num) => this.padZero(num, 8));

      //pole byte hodnot se hodi do jednoho stringu a na konec se prida 1
      let byteString = this.byteArray.join('') + '1';

      // kazdy chunk je potreba aby mel 512 bitu, krome posledniho, ten bude mit 448, aby se nakonec
      // hodila hodnota 64 bitu. Pokud delka byteStringu mod 512 neni 448, pridavej nakonec 0. 
      while (byteString.length % 512 !== 448) {
        byteString += '0';
      }

      // delka retezce pole s binarnimi hodnotami ascii znaku, 
      const length = this.byteArray.join('').length;
      //delka retezce prevedena na bin. cislo
      const binaryLength = length.toString(2);

      // maximalni delka retezce sha-1 2^64 - 1, delka se vejde na 64 bitu a pripoji se na konec byteStringu
      byteString += this.padZero(binaryLength, 64);
      this.byteString = byteString;

      // rozdeleni binaryString do bloku po 512 bitech
      this.blocks = this.chunkSubstr(this.byteString, 512);

      //kazdy blok rodelit 512 bitu rodelit po 16x32 bitech
      this.wordsOfBlocks = this.blocks.map((block) => this.chunkSubstr(block, 32));

      // pridani novych slov, z 16 na 80 (32-cifernych), aplikace XOR na vsechna (vybrana 4 slova z bloku)
      const words80 = this.wordsOfBlocks.map((block) => {
    
        for (let i = 16; i <= 79; i++) {
      
          const wordA = block[i - 3]; //vybrana 4 slova
          const wordB = block[i - 8];
          const wordC = block[i - 14];
          const wordD = block[i - 16];

          const xorA = this.xor(wordA, wordB);
          const xorB = this.xor(xorA, wordC);
          const xorC = this.xor(xorB, wordD);

          //leva rotace k prave o 1 bit 
          const leftRotated = this.leftRotate(xorC, 1);
          block.push(leftRotated);
        }
        return block;
      });

      //bitove operace na uvodni konstanty a bloky slov
      for (let i = 0; i < words80.length; i++) {
        
        let a = h0;
        let b = h1;
        let c = h2;
        let d = h3;
        let e = h4;

        for (let j = 0; j < 80; j++) {
          let f;
          let k;
          if (j < 20) {
            const BandC = this.and(b, c);
            const notB = this.and(this.not(b), d);
            f = this.or(BandC, notB);
            k = '01011010100000100111100110011001';
          }
          else if (j < 40) {
            const BxorC = this.xor(b, c);
            f = this.xor(BxorC, d);
            k = '01101110110110011110101110100001';
          }
          else if (j < 60) {
            const BandC = this.and(b, c);
            const BandD = this.and(b, d);
            const CandD = this.and(c, d);
            const BandCorBandD = this.or(BandC, BandD);
            f = this.or(BandCorBandD, CandD);
            k = '10001111000110111011110011011100';
          }
          else {
            const BxorC = this.xor(b, c);
            f = this.xor(BxorC, d);
            k = '11001010011000101100000111010110';
          }
          //konstansty se priradi a pouziji v dalsim cykly (z 80)
          const word = words80[i][j];
          const tempA = this.binaryAddition(this.leftRotate(a, 5), f);
          const tempB = this.binaryAddition(tempA, e);
          const tempC = this.binaryAddition(tempB, k);
          let temp = this.binaryAddition(tempC, word);

          temp = this.truncate(temp, 32);
          e = d;
          d = c;
          c = this.leftRotate(b, 30);
          b = a;
          a = temp;
        }

        //dohromady konstanty a zkraceni na 32 bitu
        this.h0 = this.truncate(this.binaryAddition(h0, a), 32);
        this.h1 = this.truncate(this.binaryAddition(h1, b), 32);
        this.h2 = this.truncate(this.binaryAddition(h2, c), 32);
        this.h3 = this.truncate(this.binaryAddition(h3, d), 32);
        this.h4 = this.truncate(this.binaryAddition(h4, e), 32);
      }
      
      this.hash = [this.h0, this.h1, this.h2, this.h3, this.h4].map((string) => this.binaryToHex(string)).join('');
    },

    isHash() {
      return this.hash;
    },

    padZero(num, length) {
      let numArray = num.toString().split('');
      while (numArray.length < length) {
        numArray.unshift('0');
      }

      return numArray.join('');
    },

    leftRotate(string, num) {
      return string.slice(num) + string.slice(0, num);
    },

    binaryToHex(string) {
      if (typeof string !== 'string') string = string.toString();
      let decimal = parseInt(string, 2);
      return decimal.toString(16);
    },

     chunkSubstr(str, size) {
      const numChunks = Math.ceil(str.length / size)
      const chunks = new Array(numChunks)

      for (let i = 0, o = 0; i < numChunks; ++i, o += size) {
        chunks[i] = str.substr(o, size)
      }

      return chunks
    },

    xor(strA, strB) {
      let arrayA = strA.split('').map((letter) => parseInt(letter) );
      let arrayB = strB.split('').map((letter) => parseInt(letter) );
      const xorArray = arrayA.map((num, index) => num ^ arrayB[index]);
      return xorArray.join('').toString();
    },

    and(strA, strB) {
      let arrayA = strA.split('').map((letter) => +letter );
      let arrayB = strB.split('').map((letter) => +letter );
      const andArray = arrayA.map((num, index) => num & arrayB[index]);
      return andArray.join('').toString();
    },

    or(strA, strB) {
      let arrayA = strA.split('').map((letter) => +letter );
      let arrayB = strB.split('').map((letter) => +letter );
      const orArray = arrayA.map((num, index) => num | arrayB[index]);
      return orArray.join('').toString();
    },

   not(strA) {
      let array = strA.split('').map((letter) => letter );
      return array.map(letter => {
        if (letter === '1') return '0';
        return '1';
      }).join('');
    },

    binaryAddition(stringA, stringB) {
      const numA = parseInt(stringA, 2);
      const numB = parseInt(stringB, 2);
      let sum = (numA + numB).toString(2);
      const length = stringA.length;

      while (sum.length < stringA.length) {
        sum = '0' + sum;
      }

      return sum.length === length ? '1' + sum : sum;
    },

    truncate(string, length) {
      while (string.length > length) {
        string = string.slice(1);
      }

      return string;
    },
    hexToBinary(hex) {
      const number = parseInt(hex, 16);
      return (number >>> 0).toString(2); //>>> je shift, cili 0 posuvu doleva
      
    },

    clearHash() {
      this.hash = '';
      this.message = '';
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->