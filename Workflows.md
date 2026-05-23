# Professional-calculator
          from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.gridlayout import GridLayout
from kivy.uix.button import Button
from kivy.uix.textinput import TextInput
from kivy.uix.tabbedpanel import TabbedPanel, TabbedPanelHeader
from kivy.uix.label import Label
from kivy.config import Config
from kivy.graphics import Color, RoundedRectangle
import math
import re
from collections import defaultdict

Config.set('graphics', 'width', '450')
Config.set('graphics', 'height', '750')

class StyledButton(Button):
    def __init__(self, text_color=(1, 1, 1, 1), **kwargs):
        super().__init__(**kwargs)
        self.font_size = 22
        self.bold = True
        self.background_normal = ''
        self.background_down = ''
        self.color = text_color
        
    def set_color(self, color, text_color=None):
        self.background_color = color
        if text_color:
            self.color = text_color

class CalculatorApp(App):
    def build(self):
        self.tabs = TabbedPanel(do_default_tab=False, tab_pos='top_mid')
        
        calc_tab = TabbedPanelHeader(text='Calculator')
        calc_tab.content = self.create_calculator()
        self.tabs.add_widget(calc_tab)
        
        algebra_tab = TabbedPanelHeader(text='Algebra')
        algebra_tab.content = self.create_algebra_tab()
        self.tabs.add_widget(algebra_tab)
        
        vector_tab = TabbedPanelHeader(text='Vector & Pythagoras')
        vector_tab.content = self.create_vector_tab()
        self.tabs.add_widget(vector_tab)
        
        return self.tabs
    
    def create_calculator(self):
        main_layout = BoxLayout(orientation='vertical', padding=15, spacing=10)
        
        display_box = BoxLayout(size_hint_y=0.25, padding=5)
        with display_box.canvas.before:
            Color(0.15, 0.15, 0.2, 1)
            self.rect = RoundedRectangle(pos=display_box.pos, size=display_box.size, radius=[10])
            display_box.bind(pos=lambda obj, val: setattr(self.rect, 'pos', val))
            display_box.bind(size=lambda obj, val: setattr(self.rect, 'size', val))
        
        self.display = TextInput(
            multiline=False,
            readonly=True,
            halign='right',
            font_size=36,
            background_color=(0, 0, 0, 0),
            foreground_color=(0.9, 0.9, 1, 1),
            cursor_color=(0, 0, 0, 0)
        )
        display_box.add_widget(self.display)
        main_layout.add_widget(display_box)
        
        buttons_layout = BoxLayout(orientation='vertical', spacing=8, size_hint_y=0.75)
        
        row1 = BoxLayout(spacing=8)
        funcs = ['√', 'x²', '^', '|x|', 'avg', ',']
        for func in funcs:
            btn = StyledButton(text=func, text_color=(1, 1, 1, 1))
            btn.set_color((0.2, 0.6, 0.9, 1))
            btn.bind(on_press=self.on_button_press)
            row1.add_widget(btn)
        buttons_layout.add_widget(row1)
        
        row2 = BoxLayout(spacing=8)
        specials = [
            ('C', (0.9, 0.3, 0.3, 1), (1, 1, 1, 1)),
            ('⌫', (0.9, 0.5, 0.2, 1), (1, 1, 1, 1)),
            ('(', (0.5, 0.5, 0.6, 1), (1, 1, 1, 1)),
            (')', (0.5, 0.5, 0.6, 1), (1, 1, 1, 1)),
            ('÷', (0.5, 0.5, 0.6, 1), (1, 1, 1, 1))
            ]
           
        for text, bg_color, txt_color in specials:
            btn = StyledButton(text=text, text_color=txt_color)
            btn.set_color(bg_color)
            btn.bind(on_press=self.on_button_press)
            row2.add_widget(btn)
        buttons_layout.add_widget(row2)
        
        nums = [
            ['7', '8', '9', '×'],
            ['4', '5', '6', '-'],
            ['1', '2', '3', '+']
        ]
        
        for row in nums:
            box = BoxLayout(spacing=8)
            for val in row:
                if val in ['×', '-', '+']:
                    btn = StyledButton(text=val, text_color=(1, 1, 1, 1))
                    btn.set_color((0.4, 0.8, 0.4, 1))
                else:
                    btn = StyledButton(text=val, text_color=(0, 0, 0, 1))
                    btn.set_color((0.95, 0.95, 0.95, 1))
                btn.bind(on_press=self.on_button_press)
                box.add_widget(btn)
            buttons_layout.add_widget(box)
        
        last_row = BoxLayout(spacing=8)
        zero_btn = StyledButton(text='0', text_color=(0, 0, 0, 1), size_hint_x=2)
        zero_btn.set_color((0.95, 0.95, 0.95, 1))
        zero_btn.bind(on_press=self.on_button_press)
        
        dot_btn =  StyledButton(text='.', text_color=(0, 0, 0, 1))
        dot_btn.set_color((0.95, 0.95, 0.95, 1))
        dot_btn.bind(on_press=self.on_button_press) 
        
        eq_btn = StyledButton(text='=', text_color=(1, 1, 1, 1))
        eq_btn.set_color((0.2, 0.8, 0.4, 1))
        eq_btn.bind(on_press=self.on_button_press)
        
        last_row.add_widget(zero_btn)
        last_row.add_widget(dot_btn)
        last_row.add_widget(eq_btn)
        buttons_layout.add_widget(last_row)
        
        main_layout.add_widget(buttons_layout)
        return main_layout
    
    def create_algebra_tab(self):
        layout = BoxLayout(orientation='vertical', padding=15, spacing=10)
        
        layout.add_widget(Label(
            text='Algebraic Expression Calculator',
            font_size=22,
            color=(0.2, 0.6, 0.9, 1),
            size_hint_y=0.07
        ))
        
        layout.add_widget(Label(text='Expression 1:', size_hint_y=0.05))
        self.algebra1 = TextInput(
            multiline=False,
            font_size=18,
            hint_text='e.g., 2x^2 + 3x - 5',
            background_color=(0.95, 0.95, 0.95, 1),
            foreground_color=(0, 0, 0, 1),
            size_hint_y=0.1
        )
        layout.add_widget(self.algebra1)
        
        op_box = BoxLayout(size_hint_y=0.08, spacing=10)
        self.selected_op = '+'
        self.op_buttons = {}
        
        for op, color in [('+', (0.4, 0.8, 0.4, 1)), 
                         ('-', (0.9, 0.5, 0.3, 1)),
                         ('×', (0.3, 0.6, 0.9, 1)), 
                         ('÷', (0.8, 0.4, 0.8, 1))]:
            btn = Button(text=op, background_color=color, font_size=20)
            btn.bind(on_press=lambda inst, o=op: self.select_op(inst, o))
            self.op_buttons[op] = btn
            op_box.add_widget(btn)
        layout.add_widget(op_box)
        
        layout.add_widget(Label(text='Expression 2:', size_hint_y=0.05))
        self.algebra2 = TextInput(
            multiline=False,
            font_size=18,
            hint_text='e.g., x^2 - 2x + 1',
            background_color=(0.95, 0.95, 0.95, 1),
            foreground_color=(0, 0, 0, 1),
            size_hint_y=0.1
        )
        layout.add_widget(self.algebra2)
        
        calc_btn = Button(
            text='Calculate',
            font_size=20,
            background_color=(0.2, 0.7, 0.9, 1),
            size_hint_y=0.09
        )
        calc_btn.bind(on_press=self.calculate_algebra)
        layout.add_widget(calc_btn)
        
        self.alg_result = TextInput(
            readonly=True,
            font_size=18,
            hint_text='Result will appear here...',
            background_color=(0.15, 0.2, 0.15, 1),
            foreground_color=(0, 1, 0.2, 1),
            size_hint_y=0.15
        )
        layout.add_widget(self.alg_result)
        
        help_text = '''
Tips:
• x^2 means x squared
• 2x means 2 times x
• Use * for multiplication: 2*x or 2x
• Only variable x is supported
        '''
        layout.add_widget(Label(
            text=help_text,
            font_size=13,
            color=(0.6, 0.6, 0.6, 1),
            size_hint_y=0.25
        ))
        
        return layout
    
    def select_op(self, instance, op):
        self.selected_op = op
        for btn in self.op_buttons.values():
            btn.opacity = 0.6
        instance.opacity = 1
    
    def parse_polynomial(self, expr):
        expr = expr.replace(' ', '')
        if not expr:
            return {}
        
        if expr[0] not in '+-':
            expr = '+' + expr
        
        terms = re.findall(r'[+-][^+-]+', expr)
        result = defaultdict(float)
        
        for term in terms:
            sign = 1 if term[0] == '+' else -1
            term = term[1:]
            
            if 'x' in term:
                if '^' in term:
                    parts = term.split('x^')
                    coef_part = parts[0]
                    power = int(parts[1])
                else:
                    coef_part = term.replace('x', '')
                    power = 1
                
                if coef_part == '' or coef_part == '*':
                    coef = 1
                else:
                    coef = float(coef_part.replace('*', ''))
            else:
                coef = float(term) if term else 0
                power = 0
            
            result[power] += sign * coef
        
        return dict(result)
    
    def format_polynomial(self, poly_dict):
        if not poly_dict:
            return "0"
        
        terms = []
        for power in sorted(poly_dict.keys(), reverse=True):
            coef = poly_dict[power]
            if coef == 0:
                continue
            
            if coef > 0 and terms:
                sign = " + "
            elif coef < 0:
                sign = " - "
                coef = abs(coef)
            else:
                sign = ""
            
            if coef == 1 and power != 0:
                coef_str = ""
            elif coef == int(coef):
                coef_str = str(int(coef))
            else:
                coef_str = str(coef)
            
            if power == 0:
                var_str = ""
            elif power == 1:
                var_str = "x"
            else:
                var_str = f"x^{power}"
            
            if coef_str and var_str:
                term_str = f"{coef_str}*{var_str}" if '*' not in coef_str else f"{coef_str}{var_str}"
            else:
                term_str = coef_str + var_str
            
            terms.append(sign + term_str)
        
        return "".join(terms) if terms else "0"
    
    def add_poly(self, p1, p2):
        result = defaultdict(float)
        for power, coef in p1.items():
            result[power] += coef
        for power, coef in p2.items():
            result[power] += coef
        return dict(result)
    
    def sub_poly(self, p1, p2):
        result = defaultdict(float)
        for power, coef in p1.items():
            result[power] += coef
        for power, coef in p2.items():
            result[power] -= coef
        return dict(result)
    
    def multiply_poly(self, p1, p2):
        result = defaultdict(float)
        for p1_pow, p1_coef in p1.items():
            for p2_pow, p2_coef in p2.items():
                result[p1_pow + p2_pow] += p1_coef * p2_coef
        return dict(result)
    
    def divide_poly(self, p1, p2):
        if len(p2) == 1:
            power2, coef2 = list(p2.items())[0]
            if coef2 == 0:
                return None
            result = {}
            for power, coef in p1.items():
                result[power - power2] = coef / coef2
            return result
        return None
    
    def calculate_algebra(self, instance):
        try:
            expr1 = self.algebra1.text.strip()
            expr2 = self.algebra2.text.strip()
            op = self.selected_op
            
            if not expr1 or not expr2:
                self.alg_result.text = "Error: Enter both expressions"
                return
            
            poly1 = self.parse_polynomial(expr1)
            poly2 = self.parse_polynomial(expr2)
            
            if op == '+':
                result = self.add_poly(poly1, poly2)
                result_str = self.format_polynomial(result)
            elif op == '-':
                result = self.sub_poly(poly1, poly2)
                result_str = self.format_polynomial(result)
            elif op == '×':
                result = self.multiply_poly(poly1, poly2)
                result_str = self.format_polynomial(result)
            elif op == '÷':
                result = self.divide_poly(poly1, poly2)
                if result is None:
                    result_str = "Complex polynomial division not supported"
                else:
                    result_str = self.format_polynomial(result)
            
            pretty1 = self.format_polynomial(poly1)
            pretty2 = self.format_polynomial(poly2)
            self.alg_result.text = f"({pretty1}) {op} ({pretty2})\n= {result_str}"
            
        except Exception as e:
            self.alg_result.text = f"Error: {str(e)}\nCheck format"
    
    def on_button_press(self, instance):
        text = instance.text
        current = self.display.text
        
        replacements = {'×': '*', '÷': '/'}
        text = replacements.get(text, text)
        
        if text == 'C':
            self.display.text = ''
        elif text == '⌫':
            self.display.text = current[:-1]
        elif text == '=':
            try:
                result = self.calculate(current)
                self.display.text = str(result)
            except:
                self.display.text = 'Error'
        elif text == '√':
            self.display.text += 'sqrt('
        elif text == 'x²':
            self.display.text += '**2'
        elif text == '|x|':
            self.display.text += 'abs('
        elif text == '^':
            self.display.text += '**'
        elif text == 'avg':
            self.display.text += 'avg('
        else:
            self.display.text += text
    
    def calculate(self, expression):
        expr = expression.replace('sqrt', 'math.sqrt')
        expr = expr.replace('abs', 'abs')
        
        if 'avg(' in expr:
            matches = re.findall(r'avg\(([^)]+)\)', expr)
            for match in matches:
                numbers = [float(x.strip()) for x in match.split(',')]
                avg_val = sum(numbers) / len(numbers)
                expr = expr.replace(f'avg({match})', str(avg_val))
        
        return eval(expr)
    
    def create_vector_tab(self):
        layout = BoxLayout(orientation='vertical', padding=15, spacing=10)
        
        layout.add_widget(Label(
            text='2D Vector Operations',
            font_size=22,
            color=(0.9, 0.4, 0.4, 1),
            size_hint_y=0.08
        ))
        
        vec1_box = BoxLayout(size_hint_y=0.12, spacing=10)
        vec1_box.add_widget(Label(text='Vector A:', size_hint_x=0.3))
        self.v1x = TextInput(hint_text='x1', multiline=False, input_filter='float',
                            background_color=(1, 0.95, 0.95, 1))
        self.v1y = TextInput(hint_text='y1', multiline=False, input_filter='float',
                            background_color=(1, 0.95, 0.95, 1))
        vec1_box.add_widget(self.v1x)
        vec1_box.add_widget(self.v1y)
        layout.add_widget(vec1_box)
        
        vec2_box = BoxLayout(size_hint_y=0.12, spacing=10)
        vec2_box.add_widget(Label(text='Vector B:', size_hint_x=0.3))
        self.v2x = TextInput(hint_text='x2', multiline=False, input_filter='float',
                            background_color=(0.95, 0.95, 1, 1))
        self.v2y = TextInput(hint_text='y2', multiline=False, input_filter='float',
                            background_color=(0.95, 0.95, 1, 1))
        vec2_box.add_widget(self.v2x)
        vec2_box.add_widget(self.v2y)
        layout.add_widget(vec2_box)
        
        btn_box = BoxLayout(size_hint_y=0.1, spacing=10)
        for text, color in [('Add', (0.4, 0.8, 0.4, 1)), 
                           ('Subtract', (0.9, 0.5, 0.3, 1))]:
            btn = Button(text=text, background_color=color, font_size=18)
            btn.bind(on_press=self.vector_add if 'Add' in text else self.vector_sub)
            btn_box.add_widget(btn)
        layout.add_widget(btn_box)
        
        self.vec_result = Label(
            text='Result: ',
            font_size=20,
            color=(0.2, 0.6, 0.8, 1),
            size_hint_y=0.1
        )
        layout.add_widget(self.vec_result)
        
        layout.add_widget(Label(text='─'*30, size_hint_y=0.05))
        layout.add_widget(Label(
            text='Pythagorean Theorem',
            font_size=20,
            color=(0.6, 0.4, 0.8, 1),
            size_hint_y=0.08
        ))
        
        pyth_box = BoxLayout(size_hint_y=0.12, spacing=10)
        self.pyth_a = TextInput(hint_text='a', input_filter='float',
                               background_color=(0.95, 1, 0.95, 1))
        self.pyth_b = TextInput(hint_text='b', input_filter='float',
                               background_color=(0.95, 1, 0.95, 1))
        self.pyth_c = TextInput(hint_text='c (optional)', input_filter='float',
                               background_color=(0.95, 1, 0.95, 1))
        pyth_box.add_widget(self.pyth_a)
        pyth_box.add_widget(self.pyth_b)
        pyth_box.add_widget(self.pyth_c)
        layout.add_widget(pyth_box)
        
        pyth_btn = Button(
            text='Calculate',
            background_color=(0.6, 0.4, 0.8, 1),
            size_hint_y=0.1
        )
        pyth_btn.bind(on_press=self.pythagoras_calc)
        layout.add_widget(pyth_btn)
        
        self.pyth_result = Label(
            text='Result: ',
            font_size=18,
            color=(0.4, 0.7, 0.4, 1),
            size_hint_y=0.1
        )
        layout.add_widget(self.pyth_result)
        
        return layout
    
    def vector_add(self, instance):
        try:
            x1 = float(self.v1x.text or 0)
            y1 = float(self.v1y.text or 0)
            x2 = float(self.v2x.text or 0)
            y2 = float(self.v2y.text or 0)
            self.vec_result.text = f'A + B = ({x1 + x2:.2f}, {y1 + y2:.2f})'
        except:
            self.vec_result.text = 'Input Error'
    
    def vector_sub(self, instance):
        try:
            x1 = float(self.v1x.text or 0)
            y1 = float(self.v1y.text or 0)
            x2 = float(self.v2x.text or 0)
            y2 = float(self.v2y.text or 0)
            self.vec_result.text = f'A - B = ({x1 - x2:.2f}, {y1 - y2:.2f})'
        except:
            self.vec_result.text = 'Input Error'
    
    def pythagoras_calc(self, instance):
        try:
            a = float(self.pyth_a.text or 0)
            b = float(self.pyth_b.text or 0)
            c_text = self.pyth_c.text
            
            if c_text:
                c = float(c_text)
                if a == 0 and b != 0:
                    a = math.sqrt(abs(c**2 - b**2))
                    self.pyth_result.text = f'a = {a:.4f}'
                elif b == 0 and a != 0:
                    b = math.sqrt(abs(c**2 - a**2))
                    self.pyth_result.text = f'b = {b:.4f}'
                else:
                    self.pyth_result.text = 'Set a or b to zero'
            else:
                c = math.sqrt(a**2 + b**2)
                self.pyth_result.text = f'c (hypotenuse) = {c:.4f}'
        except:
            self.pyth_result.text = 'Calculation Error'

if __name__ == '__main__':
    CalculatorApp().run()
    

name: Build Android APK

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Grant execute permission for gradlew
      run: chmod +x ./gradlew

    - name: Build Debug APK
      run: ./gradlew assembleDebug

    - name: Upload APK
      uses: actions/upload-artifact@v4
      with:
        name: app-debug-apk
        path: app/build/outputs/apk/debug/app-debug.apk
