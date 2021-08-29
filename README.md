import kivy
from kivy.lang import Builder
from kivy.app import App
from kivy.uix.label import Label
from kivy.uix.gridlayout import GridLayout
from kivy.uix.textinput import TextInput
from kivy.uix.button import Button
from kivy.properties import StringProperty
from kivy.uix.widget import Widget
from kivy.uix.screenmanager import ScreenManager, Screen
from kivy_garden.graph import Graph, MeshLinePlot

from pandas_datareader import data
import pandas as pd

import matplotlib.pyplot as plt
from kivy.garden.matplotlib.backend_kivyagg import FigureCanvasKivyAgg

from bs4 import BeautifulSoup
import requests
import bs4
import yfinance as yf

from urllib.request import urlretrieve as retrieve
from urllib.request import urlopen
import csv
import re
import os


kivy.require("1.10.1")

Xcolor = [0, 3, 2, 1]

plt.style.use('dark_background')


Builder.load_string("""
<menuscreen>:
    AnchorLayout:
        anchor_x: 'right'
        anchor_y: 'top'
        Button:
            text: 'Exit'
            size_hint: .1, .05
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: app.stop()
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
    AnchorLayout:
        anchor_x: 'left'
        anchor_y: 'top'
        Button:
            text: 'Market Indicators'
            size_hint: .1, .05
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.djndsp()
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
    BoxLayout:
        orientation: 'vertical'
        Label:
            text: 'Menu'
            font_size: '50sp'
            color: 0, 3, 2, 1
        Label:
            text: ' '
        Label:
            text: ' '
        BoxLayout:
            orientation: 'horizontal'
            Label:
                text: ' '
            Label:
                id: dj
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: djprice
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: djchg
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                id: nd
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: ndprice
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: ndchg
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                id: sp
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: sp500price
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: sp500chg
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                text: ' '
        Label:
            text: ' '
        Label:
            text: ' '
        Label:
            text: ' '
        Label:
            text: ' '
        Label:
            text: ' '
        Label:
            text: ' '
        Label:
            text: ' '
    FloatLayout:
        Button:
            text: 'Search'
            font_size: '30sp'
            size_hint: .2, .1
            pos: 400, 400
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'search'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 4
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'Stock/ETF Analysis'
            font_size: '30sp'
            size_hint: .2, .1
            pos: 400, 200
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'curan'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 4
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'Spy'
            font_size: '30sp'
            size_hint: .2, .1
            pos: 1200, 400
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'spy'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 4
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'Your List'
            font_size: '30sp'
            size_hint: .2, .1
            pos: 1200, 200
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'list'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 4
                    rectangle: self.x, self.y, self.width, self.height

<searchscreen>:
    AnchorLayout:
        anchor_x: 'left'
        anchor_y: 'bottom'
        Button:
            text: 'Back'
            size_hint: .1, .05
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'menu'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
    BoxLayout:
        orientation: 'vertical'
        BoxLayout:
            orientation: 'horizontal'
            Image:
                id: onedimg
                source: '1dpic.png'
            Image:
                id: onewimg
                source: '1wpic.png'
            Image:
                id: onemimg
                source: '1mpic.png'
            Image:
                id: threemimg
                source: '3mpic.png'
        BoxLayout:
            orientation: 'horizontal'
            Image:
                id: sixmimg
                source: '6mpic.png'
            Image:
                id: oneyimg
                source: '1ypic.png'
            Image:
                id: twoyimg
                source: '2ypic.png'
            Image:
                id: fvyimg
                source: '5ypic.png'
        BoxLayout:
            orientation: 'horizontal'
            BoxLayout:
                orientation: 'horizontal'
                BoxLayout:
                    orientation: 'vertical'
                    Label:
                        text: ' '
                BoxLayout:
                    orientation: 'vertical'
                    Label:
                        text: ' '
                    Label:
                        text: ' '
                    Label:
                        id: searchlbl1
                        text: ' '
                        font_size: '15sp'
                        color: 0, 3, 2, 1
                    Label:
                        id: searchlbl2
                        text: ' '
                        font_size: '15sp'
                        color: 0, 3, 2, 1
                    Label:
                        id: searchlbl3
                        text: ' '
                        font_size: '15sp'
                        color: 0, 3, 2, 1
                    Label:
                        id: searchlbl4
                        text: ' '
                        font_size: '15sp'
                        color: 0, 3, 2, 1
                    Label:
                        id: searchlbl5
                        text: ' '
                        font_size: '15sp'
                        color: 0, 3, 2, 1
                    Label:
                        id: searchlbl6
                        text: ' '
                        font_size: '15sp'
                        color: 0, 3, 2, 1
                    Label:
                        id: searchlbl7
                        text: ' '
                        font_size: '15sp'
                        color: 0, 3, 2, 1
                    Label:
                        id: searchlbl8
                        text: ' '
                        font_size: '15sp'
                        color: 0, 3, 2, 1
                    Label:
                        id: searchlbl9
                        text: ' '
                        font_size: '15sp'
                        color: 0, 3, 2, 1
            BoxLayout:
                orientation: 'vertical'
                Label:
                    text: ' '
                Label:
                    text: ' '
                Label:
                    id: xsearchlbl1
                    text: ' '
                    font_size: '15sp'
                    color: 0, 3, 2, 1
                Label:
                    id: xsearchlbl2
                    text: ' '
                    font_size: '15sp'
                    color: 0, 3, 2, 1
                Label:
                    id: xsearchlbl3
                    text: ' '
                    font_size: '15sp'
                    color: 0, 3, 2, 1
                Label:
                    id: xsearchlbl4
                    text: ' '
                    font_size: '15sp'
                    color: 0, 3, 2, 1
                Label:
                    id: xsearchlbl5
                    text: ' '
                    font_size: '15sp'
                    color: 0, 3, 2, 1
                Label:
                    id: xsearchlbl6
                    text: ' '
                    font_size: '15sp'
                    color: 0, 3, 2, 1
                Label:
                    id: xsearchlbl7
                    text: ' '
                    font_size: '15sp'
                    color: 0, 3, 2, 1
                Label:
                    id: xsearchlbl8
                    text: ' '
                    font_size: '15sp'
                    color: 0, 3, 2, 1
                Label:
                    id: xsearchlbl9
                    text: ' '
                    font_size: '15sp'
                    color: 0, 3, 2, 1
            BoxLayout:
                orientation: 'vertical'
                Label:
                    id: searchlabel
                    text: ' '
                    font_size: '20sp'
                    color: 0, 3, 2, 1
                TextInput:
                    id: inputstock
                    size_hint_x: 4
                    font_size: '30sp'
                    foreground_color: 0, 3, 2, 1
                    background_color: 0, 0, 0, 1
                    canvas.before:
                        Color:
                            rgba: 0, 3, 2, 1
                        Line:
                            width: 2
                            rectangle: self.x, self.y, self.width, self.height
                BoxLayout:
                    orientation: 'horizontal'
                    Button:
                        text: 'Stock'
                        font_size: '30sp'
                        color: 0, 3, 2, 1
                        background_color: 0, 0, 0, 1
                        on_press: root.stocksearchref()
                        on_press: root.onedgraph()
                        on_press: root.onewgraph()
                        on_press: root.onemgraph()
                        on_press: root.threemgraph()
                        on_press: root.sixmgraph()
                        on_press: root.oneygraph()
                        on_press: root.twoygraph()
                        on_press: root.fvygraph()
                        canvas.before:
                            Color:
                                rgba: 0, 3, 2, 1
                            Line:
                                width: 2
                                rectangle: self.x, self.y, self.width, self.height
                    Button:
                        text: 'ETF'
                        font_size: '30sp'
                        color: 0, 3, 2, 1
                        background_color: 0, 0, 0, 1
                        on_press: root.etfsearchref()
                        on_press: root.onedgraph()
                        on_press: root.onewgraph()
                        on_press: root.onemgraph()
                        on_press: root.threemgraph()
                        on_press: root.sixmgraph()
                        on_press: root.oneygraph()
                        on_press: root.twoygraph()
                        on_press: root.fvygraph()
                        canvas.before:
                            Color:
                                rgba: 0, 3, 2, 1
                            Line:
                                width: 2
                                rectangle: self.x, self.y, self.width, self.height

<spyscreen>:
    AnchorLayout:
        anchor_x: 'right'
        anchor_y: 'top'
        Button:
            text: 'Back'
            size_hint: .1, .05
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'menu'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
    BoxLayout:
        orientation: 'vertical'
        Label:
            text: 'Spy'
            font_size: '50sp'
            color: 0, 3, 2, 1
        Label:
            text: ' '
        Label:
            text: ' '
        Label:
            text: ' '
        Button:
            text: 'Spy on Hedgefunds'
            font_size: '30sp'
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'hedge'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'Spy on Senators'
            font_size: '30sp'
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'pol'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 5
                    rectangle: self.x, self.y, self.width, self.height

<listscreen>:
    AnchorLayout:
        anchor_x: 'right'
        anchor_y: 'top'
        Button:
            text: 'Back'
            size_hint: .1, .05
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'menu'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
    AnchorLayout:
        anchor_x: 'left'
        anchor_y: 'top'
        Button:
            text: 'Run/Refresh'
            size_hint: .1, .05
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.listref()
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
    FloatLayout:
        Label:
            text: 'Input Ticker:'
            color: 0, 3, 2, 1
            size_hint: .05, .025
            pos: 800, 970
        TextInput:
            id: inputlist
            size_hint: .05, .025
            pos: 900, 970
            font_size: '15sp'
            foreground_color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 1
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'Add'
            size_hint: .05, .025
            pos: 1000, 970
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.addlist()
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
    FloatLayout:
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 900
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.delist1()
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 877.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 855
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 832.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 810
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 787.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 765
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 742.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 720
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 697.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 675
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 652.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 630
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 607.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 585
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 562.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 540
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 517.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 495
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 472.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 450
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 427.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 405
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 382.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 360
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 337.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 315
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 292.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 270
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 247.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 225
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 202.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 180
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 157.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 135
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 112.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 90
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 67.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 45
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 22.5
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'X'
            size_hint: .015, .015
            pos: 0, 0
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: .5
                    rectangle: self.x, self.y, self.width, self.height
    BoxLayout:
        orientation: 'horizontal'
        BoxLayout:
            orientation: 'vertical'
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                id: listlbl1
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl2
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl3
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl4
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl5
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl6
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl7
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl8
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl9
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl10
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl11
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl12
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl13
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl14
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl15
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl16
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl17
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl18
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl19
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl20
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl21
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl22
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl23
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl24
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl25
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl26
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl27
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl28
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl29
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl30
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl31
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl32
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl33
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl34
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl35
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl36
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl37
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl38
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl39
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listlbl40
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
        BoxLayout:
            orientation: 'vertical'
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                id: listnamelbl1
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl2
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl3
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl4
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl5
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl6
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl7
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl8
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl9
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl10
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl11
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl12
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl13
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl14
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl15
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl16
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl17
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl18
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl19
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl20
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl21
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl22
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl23
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl24
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl25
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl26
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl27
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl28
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl29
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl30
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl31
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl32
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl33
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl34
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl35
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl36
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl37
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl38
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl39
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listnamelbl40
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
        BoxLayout:
            orientation: 'vertical'
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                id: listpricelbl1
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl2
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl3
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl4
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl5
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl6
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl7
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl8
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl9
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl10
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl11
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl12
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl13
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl14
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl15
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl16
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl17
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl18
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl19
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl20
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl21
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl22
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl23
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl24
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl25
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl26
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl27
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl28
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl29
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl30
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl31
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl32
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl33
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl34
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl35
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl36
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listpricelbl37
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1 
            Label:
                id: listpricelbl38
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1 
            Label:
                id: listpricelbl39
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1 
            Label:
                id: listpricelbl40
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1 
        BoxLayout:
            orientation: 'vertical'
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                id: listchglbl1
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl2
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl3
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl4
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl5
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl6
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl7
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl8
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl9
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl10
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl11
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl12
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl13
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl14
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl15
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl16
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl17
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl18
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl19
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl20
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl21
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl22
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl23
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl24
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl25
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl26
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl27
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl28
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl29
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl30
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl31
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl32
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl33
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl34
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl35
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl36
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl37
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl38
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl39
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: listchglbl40
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1

<curanscreen>:
    AnchorLayout:
        anchor_x: 'right'
        anchor_y: 'top'
        Button:
            text: 'Back'
            size_hint: .1, .05
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'menu'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
    FloatLayout:
        Button:
            text: 'Top Gainers'
            size_hint: .12, .06
            pos: 350, 920
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.gainref()
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'Biggest Losers'
            size_hint: .12, .06
            pos: 650, 920
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.losref()
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'Most Active'
            size_hint: .12, .06
            pos: 950, 920
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.activeref()
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
        Button:
            text: 'Most Volatile'
            size_hint: .12, .06
            pos: 1250, 920
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.volaref()
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
    BoxLayout:
        orientation: 'horizontal'
        BoxLayout:
            orientation: 'vertical'
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                id: q
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: w
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: e
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: r
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: t
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: y
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: u
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: i
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: o
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: p
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: a
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: s
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: d
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: f
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: g
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: h
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: j
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: k
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: l
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: z
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: x
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: c
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: v
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: b
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: n
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
        BoxLayout:
            orientation: 'vertical' 
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                id: gainlbl0
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl1
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:            
                id: gainlbl2
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl3
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl4
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl5
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl6
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl7
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl8
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl9
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl10
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl11
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl12
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl13
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl14
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl15
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl16
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl17
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl18
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl19
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl20
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl21
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl22
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl23
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl24
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
        BoxLayout:
            orientation: 'vertical'
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                id: gainlblrt0
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt1
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt2
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt3
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt4
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt5
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt6
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt7
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt8
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt9
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt10
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt11
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt12
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt13
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt14
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt15
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt16
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt17
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt18
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt19
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt20
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt21
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt22
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt23
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt24
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
        BoxLayout:
            orientation: 'vertical'
            Label:
                text: ' '
        BoxLayout:
            orientation: 'vertical'
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                id: qq
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: ww
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: ee
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: rr
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: tt
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: yy
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: uu
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: ii
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: oo
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: pp
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: aa
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: ss
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: dd
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: ff
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gg
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: hh
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: jj
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: kk
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: ll
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: zz
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: xx
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: cc
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: vv
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: bb
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: nn
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
        BoxLayout:
            orientation: 'vertical'
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                id: gainlbl25
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl26
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:            
                id: gainlbl27
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl28
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl29
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl30
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl31
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl32
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl33
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl34
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl35
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl36
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl37
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl38
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl39
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl40
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl41
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl42
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl43
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl44
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl45
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl46
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl47
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl48
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlbl49
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
        BoxLayout:
            orientation: 'vertical'
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                text: ' '
            Label:
                id: gainlblrt25
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt26
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt27
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt28
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt29
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt30
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt31
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt32
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt33
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt34
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt35
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt36
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt37
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt38
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt39
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt40
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt41
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt42
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt43
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt44
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt45
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt46
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt47
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt48
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1
            Label:
                id: gainlblrt49
                text: ' '
                font_size: '15sp'
                color: 0, 3, 2, 1

<hedgescreen>:
    AnchorLayout:
        anchor_x: 'right'
        anchor_y: 'top'
        Button:
            text: 'Back'
            size_hint: .1, .05
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'spy'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
    BoxLayout:
        orientation: 'vertical'
        Label:
            text: ' '
        BoxLayout:
            orientation: 'vertical'
            Button:
                text: 'Search Specific Ticker Ownership'
                font_size: '30sp'
                color: 0, 3, 2, 1
                background_color: 0, 0, 0, 1
                on_press: root.manager.current = 'hedgesearch'
                canvas.before:
                    Color:
                        rgba: 0, 3, 2, 1
                    Line:
                        width: 2
                        rectangle: self.x, self.y, self.width, self.height
            Button:
                text: 'Latest Hedgefund Exchanges'
                font_size: '30sp'
                color: 0, 3, 2, 1
                background_color: 0, 0, 0, 1
                on_press: root.manager.current = 'hedgehold'
                canvas.before:
                    Color:
                        rgba: 0, 3, 2, 1
                    Line:
                        width: 2
                        rectangle: self.x, self.y, self.width, self.height

<polscreen>:
    AnchorLayout:
        anchor_x: 'right'
        anchor_y: 'top'
        Button:
            text: 'Back'
            size_hint: .1, .05
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'spy'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
    AnchorLayout:
        anchor_x: 'left'
        anchor_y: 'top'
        Button:
            text: 'Run/Refresh'
            size_hint: .1, .05
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.polref()
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height
    BoxLayout:
        orientation: 'horizontal'
        BoxLayout:
            orientation: 'vertical'
            Label:
                id: poldatelbl
                font_size: '13sp'
                color: 0, 3, 2, 1
        BoxLayout:
            orientation: 'vertical'
            Label:
                id: polsenlbl
                font_size: '13sp'
                color: 0, 3, 2, 1
        BoxLayout:
            orientation: 'vertical'
            Label:
                id: polnamelbl
                font_size: '13sp'
                color: 0, 3, 2, 1
        BoxLayout:
            orientation: 'vertical'
            Label:
                id: poltypelbl
                font_size: '13sp'
                color: 0, 3, 2, 1
        BoxLayout:
            orientation: 'vertical'
            Label:
                id: polassetlbl
                font_size: '13sp'
                color: 0, 3, 2, 1

<hedgeholdscreen>:
    AnchorLayout:
        anchor_x: 'right'
        anchor_y: 'top'
        Button:
            text: 'Back'
            size_hint: .1, .05
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'hedge'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height

<hedgesearchscreen>:
    AnchorLayout:
        anchor_x: 'right'
        anchor_y: 'top'
        Button:
            text: 'Back'
            size_hint: .1, .05
            color: 0, 3, 2, 1
            background_color: 0, 0, 0, 1
            on_press: root.manager.current = 'hedge'
            canvas.before:
                Color:
                    rgba: 0, 3, 2, 1
                Line:
                    width: 2
                    rectangle: self.x, self.y, self.width, self.height

""")

class menuscreen(Screen):
    pass

    def djndsp(self):

        urldj = requests.get('https://finance.yahoo.com/quote/%5EDJI?p=^DJI&.tsrc=fin-srch')
        soupdj = bs4.BeautifulSoup(urldj.text, features="html.parser")
        djiprice = soupdj.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text     
        self.ids.djprice.text = str(djiprice)
        try:
            try:
                djchgp = soupdj.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                self.ids.djchg.text = str(djchgp)
            except AttributeError:
                djchgn = soupdj.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                self.ids.djchg.text = str(djchgn)
        except AttributeError:
            djchg = soupdj.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
            self.ids.djchg.text = str(djchg)
        self.ids.dj.text = 'Dow Jones:'

        urlnd = requests.get('https://finance.yahoo.com/quote/%5EIXIC?p=^IXIC&.tsrc=fin-srch')
        soupnd = bs4.BeautifulSoup(urlnd.text, features="html.parser")
        ndprice = soupnd.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text       
        self.ids.ndprice.text = str(ndprice)
        try:
            try:
                ndchgp = soupnd.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                self.ids.ndchg.text = str(ndchgp)
            except AttributeError:
                ndchgn = soupnd.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                self.ids.ndchg.text = str(ndchgn)
        except AttributeError:
            ndchg = soupnd.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
            self.ids.sp500chg.text = str(ndchg)
        self.ids.nd.text = 'NASDAQ:'

        urlsp = requests.get('https://finance.yahoo.com/quote/%5EGSPC?p=%5EGSPC')
        soupsp = bs4.BeautifulSoup(urlsp.text, features="html.parser")
        spprice = soupsp.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text       
        self.ids.sp500price.text = str(spprice)
        try:
            try:
                spchgp = soupsp.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                self.ids.sp500chg.text = str(spchgp)
            except AttributeError:
                spchgn = soupsp.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                self.ids.sp500chg.text = str(spchgn)
        except AttributeError:
            spchg = soupsp.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
            self.ids.sp500chg.text = str(spchg)
        self.ids.sp.text = 'S&P 500:'

class searchscreen(Screen):
    pass

    def stocksearchref(self):

        searchticker = self.ids.inputstock.text
        urlsearch = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (searchticker,searchticker))
        soupsearch = bs4.BeautifulSoup(urlsearch.text, features="html.parser")

        tickname = soupsearch.find('h1',{'class': 'D(ib) Fz(18px)'})

        searchprice = soupsearch.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
        searchpreclose = soupsearch.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[0].find('span').text
        searchopen = soupsearch.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[1].find('span').text
        searchvol = soupsearch.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[6].find('span').text
        searchavgvol = soupsearch.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[7].find('span').text
        searchmktcap = soupsearch.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[8].find('span').text
        searchpe = soupsearch.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[10].find('span').text
        searcheps = soupsearch.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[11].find('span').text

        try:
            try:
                searchpricechgp = soupsearch.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                self.ids.xsearchlbl2.text = str(searchpricechgp)
            except AttributeError:
                searchpricechgn = soupsearch.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                self.ids.xsearchlbl2.text = str(searchpricechgn)
        except AttributeError:
            searchpricechg = soupsearch.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
            self.ids.xsearchlbl2.text = str(searchpricechg)

        self.ids.searchlabel.text = str(tickname)

        self.ids.xsearchlbl1.text = str(searchprice)
        self.ids.xsearchlbl3.text = str(searchpreclose)
        self.ids.xsearchlbl4.text = str(searchopen)
        self.ids.xsearchlbl5.text = str(searchmktcap)
        self.ids.xsearchlbl6.text = str(searchpe)
        self.ids.xsearchlbl7.text = str(searcheps)
        self.ids.xsearchlbl8.text = str(searchvol)
        self.ids.xsearchlbl9.text = str(searchavgvol)

        self.ids.searchlbl1.text = 'Price:'
        self.ids.searchlbl2.text = 'Price Change:'
        self.ids.searchlbl3.text = 'Previous Close:'
        self.ids.searchlbl4.text = 'Open:'
        self.ids.searchlbl5.text = 'Market Cap:'
        self.ids.searchlbl6.text = 'P/E Ratio:'
        self.ids.searchlbl7.text = 'EPS:'
        self.ids.searchlbl8.text = 'Volume:'
        self.ids.searchlbl9.text = 'Average Volume:'

    def etfsearchref(self):

        searchetf = self.ids.inputstock.text
        urlsearchetf = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (searchetf,searchetf))
        soupsearchetf = bs4.BeautifulSoup(urlsearchetf.text, features="html.parser")

        etfname = soupsearchetf.find('h1',{'class': 'D(ib) Fz(18px)'}).text

        esearchprice = soupsearchetf.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
        esearchpreclose = soupsearchetf.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[0].find('span').text
        esearchopen = soupsearchetf.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[1].find('span').text
        esearchvol = soupsearchetf.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[6].find('span').text
        esearchavgvol = soupsearchetf.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[7].find('span').text
        esearchnetassets = soupsearchetf.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[8].find('span').text
        esearchpe = soupsearchetf.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[10].find('span').text
        esearchnav = soupsearchetf.find_all("td",{'class': 'Ta(end) Fw(600) Lh(14px)'})[9].find('span').text

        try:
            try:
                searchetfpricechgp = soupsearchetf.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                self.ids.xsearchlbl2.text = str(searchetfpricechgp)
            except AttributeError:
                searchetfpricechgn = soupsearchetf.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                self.ids.xsearchlbl2.text = str(searchetfpricechgn)
        except AttributeError:
            searchetfpricechg = soupsearchetf.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
            self.ids.xsearchlbl2.text = str(searchetfpricechg)

        self.ids.searchlabel.text = str(etfname)

        self.ids.xsearchlbl1.text = str(esearchprice)
        self.ids.xsearchlbl3.text = str(esearchpreclose)
        self.ids.xsearchlbl4.text = str(esearchopen)
        self.ids.xsearchlbl5.text = str(esearchnetassets)
        self.ids.xsearchlbl6.text = str(esearchnav)
        self.ids.xsearchlbl7.text = str(esearchpe)
        self.ids.xsearchlbl8.text = str(esearchvol)
        self.ids.xsearchlbl9.text = str(esearchavgvol)

        self.ids.searchlbl1.text = 'Price:'
        self.ids.searchlbl2.text = 'Price Change:'
        self.ids.searchlbl3.text = 'Previous Close:'
        self.ids.searchlbl4.text = 'Open:'
        self.ids.searchlbl5.text = 'Net Assets:'
        self.ids.searchlbl6.text = 'Net Asset Value:'
        self.ids.searchlbl7.text = 'P/E Ratio:'
        self.ids.searchlbl8.text = 'Volume:'
        self.ids.searchlbl9.text = 'Average Volume:'

    def onedgraph(self):

        #matplotlib<3.3.0
        plt.clf()
        searchticker = self.ids.inputstock.text
        oneddata = yf.download(tickers = searchticker,
                           period = "1d",
                           interval = "5m",
                           auto_adjust = True)
        oneddata.head()
        fig, ax = plt.subplots()
        ax.set_facecolor('black')
        oneddata['Close'].plot(color='c')
        plt.title('One Day', color='c')
        plt.xlabel('Date/Time', color='cyan')
        plt.ylabel('Price', color='cyan')
        ax.spines['bottom'].set_color('cyan')
        ax.spines['top'].set_color('cyan')
        ax.spines['left'].set_color('cyan')
        ax.spines['right'].set_color('cyan')
        ax.tick_params(axis='x', colors='cyan')
        ax.tick_params(axis='y', colors='cyan')
        plt.savefig("1dpic")
        self.ids.onedimg.reload()
        #os.remove('1dpic.png')

    def onewgraph(self):

        #matplotlib<3.3.0
        plt.clf()
        searchticker = self.ids.inputstock.text
        onewdata = yf.download(tickers = searchticker,
                           period = "5d",
                           interval = "15m",
                           auto_adjust = True)
        onewdata.head()
        fig, ax = plt.subplots()
        ax.set_facecolor('black')
        onewdata['Close'].plot(color='c')
        plt.title('One Week', color='c')
        plt.xlabel('Date/Time', color='cyan')
        plt.ylabel('Price', color='cyan')
        ax.spines['bottom'].set_color('cyan')
        ax.spines['top'].set_color('cyan')
        ax.spines['left'].set_color('cyan')
        ax.spines['right'].set_color('cyan')
        ax.tick_params(axis='x', colors='cyan')
        ax.tick_params(axis='y', colors='cyan')
        plt.savefig("1wpic")
        self.ids.onewimg.reload()
        #os.remove('1wpic.png')

    def onemgraph(self):

        #matplotlib<3.3.0
        plt.clf()
        searchticker = self.ids.inputstock.text
        onemdata = yf.download(tickers = searchticker,
                           period = "1mo",
                           interval = "30m",
                           auto_adjust = True)       
        onemdata.head()
        fig, ax = plt.subplots()
        ax.set_facecolor('black')
        onemdata['Close'].plot(color='c')
        plt.title('One Month', color='c')
        plt.xlabel('Date/Time', color='cyan')
        plt.ylabel('Price', color='cyan')
        ax.spines['bottom'].set_color('cyan')
        ax.spines['top'].set_color('cyan')
        ax.spines['left'].set_color('cyan')
        ax.spines['right'].set_color('cyan')
        ax.tick_params(axis='x', colors='cyan')
        ax.tick_params(axis='y', colors='cyan')
        plt.savefig("1mpic")
        self.ids.onemimg.reload()
        #os.remove('1mpic.png')

    def threemgraph(self):

        #matplotlib<3.3.0
        plt.clf()
        searchticker = self.ids.inputstock.text
        threemdata = yf.download(tickers = searchticker,
                           period = "3mo",
                           interval = "1d",
                           auto_adjust = True)       
        threemdata.head()
        fig, ax = plt.subplots()
        ax.set_facecolor('black')
        threemdata['Close'].plot(color='c')
        plt.title('Three Month', color='c')
        plt.xlabel('Date/Time', color='cyan')
        plt.ylabel('Price', color='cyan')
        ax.spines['bottom'].set_color('cyan')
        ax.spines['top'].set_color('cyan')
        ax.spines['left'].set_color('cyan')
        ax.spines['right'].set_color('cyan')
        ax.tick_params(axis='x', colors='cyan')
        ax.tick_params(axis='y', colors='cyan')
        plt.savefig("3mpic")
        self.ids.threemimg.reload()
        #os.remove('3mpic.png')

    def sixmgraph(self):

        #matplotlib<3.3.0
        plt.clf()
        searchticker = self.ids.inputstock.text
        sixmdata = yf.download(tickers = searchticker,
                           period = "6mo",
                           interval = "1d",
                           auto_adjust = True)       
        sixmdata.head()
        fig, ax = plt.subplots()
        ax.set_facecolor('black')
        sixmdata['Close'].plot(color='c')
        plt.title('Six Month', color='c')
        plt.xlabel('Date/Time', color='cyan')
        plt.ylabel('Price', color='cyan')
        ax.spines['bottom'].set_color('cyan')
        ax.spines['top'].set_color('cyan')
        ax.spines['left'].set_color('cyan')
        ax.spines['right'].set_color('cyan')
        ax.tick_params(axis='x', colors='cyan')
        ax.tick_params(axis='y', colors='cyan')
        plt.savefig("6mpic")
        self.ids.sixmimg.reload()
        #os.remove('6mpic.png')

    def oneygraph(self):

        #matplotlib<3.3.0
        plt.clf()
        searchticker = self.ids.inputstock.text
        oneydata = yf.download(tickers = searchticker,
                           period = "1y",
                           interval = "1d",
                           auto_adjust = True)       
        oneydata.head()
        fig, ax = plt.subplots()
        ax.set_facecolor('black')
        oneydata['Close'].plot(color='c')
        plt.title('One Year', color='c')
        plt.xlabel('Date/Time', color='cyan')
        plt.ylabel('Price', color='cyan')
        ax.spines['bottom'].set_color('cyan')
        ax.spines['top'].set_color('cyan')
        ax.spines['left'].set_color('cyan')
        ax.spines['right'].set_color('cyan')
        ax.tick_params(axis='x', colors='cyan')
        ax.tick_params(axis='y', colors='cyan')
        plt.savefig("1ypic")
        self.ids.oneyimg.reload()
        #os.remove('1ypic.png')

    def twoygraph(self):

        #matplotlib<3.3.0
        plt.clf()
        searchticker = self.ids.inputstock.text
        twoydata = yf.download(tickers = searchticker,
                           period = "2y",
                           interval = "5d",
                           auto_adjust = True)       
        twoydata.head()
        fig, ax = plt.subplots()
        ax.set_facecolor('black')
        twoydata['Close'].plot(color='c')
        plt.title('Two Year', color='c')
        plt.xlabel('Date/Time', color='cyan')
        plt.ylabel('Price', color='cyan')
        ax.spines['bottom'].set_color('cyan')
        ax.spines['top'].set_color('cyan')
        ax.spines['left'].set_color('cyan')
        ax.spines['right'].set_color('cyan')
        ax.tick_params(axis='x', colors='cyan')
        ax.tick_params(axis='y', colors='cyan')
        plt.savefig("2ypic")
        self.ids.twoyimg.reload()
        #os.remove('2ypic.png')

    def fvygraph(self):

        #matplotlib<3.3.0
        plt.clf()
        searchticker = self.ids.inputstock.text
        fvydata = yf.download(tickers = searchticker,
                           period = "5y",
                           interval = "5d",
                           auto_adjust = True)       
        fvydata.head()
        fig, ax = plt.subplots()
        ax.set_facecolor('black')
        fvydata['Close'].plot(color='c')
        plt.title('Five Year', color='c')
        plt.xlabel('Date/Time', color='cyan')
        plt.ylabel('Price', color='cyan')
        ax.spines['bottom'].set_color('cyan')
        ax.spines['top'].set_color('cyan')
        ax.spines['left'].set_color('cyan')
        ax.spines['right'].set_color('cyan')
        ax.tick_params(axis='x', colors='cyan')
        ax.tick_params(axis='y', colors='cyan')
        plt.savefig("5ypic")
        self.ids.fvyimg.reload()
        #os.remove('5ypic.png')

class spyscreen(Screen):
    pass

class listscreen(Screen):
    pass

    def listref(self):
        slistfile = open('Stocklist.csv')
        slist = csv.reader(slistfile)
        istlist = list(slist)
        dflist = pd.read_csv('stocklist.csv')
        modifieddf = dflist.dropna()
        modifieddf.to_csv('stocklist.csv',index=False)

        try:
            list1 = str(istlist[1])[2:-2]
            urllist1 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list1,list1))
            souplist1 = bs4.BeautifulSoup(urllist1.text, features="html.parser")
            list1name = souplist1.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list1price = souplist1.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list1chgp = souplist1.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl1.text = str(list1chgp)
                except AttributeError:
                    list1chgn = souplist1.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl1.text = str(list1chgn)
            except AttributeError:
                list1chg = souplist1.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl1.text = str(list1chg)
            self.ids.listnamelbl1.text = str(list1name)
            self.ids.listlbl1.text = str(list1)
            self.ids.listpricelbl1.text = str(list1price)
        except (IndexError, AttributeError):
            self.ids.listlbl1.text = str(' ')
            self.ids.listnamelbl1.text = str(' ')
            self.ids.listpricelbl1.text = str(' ')
            self.ids.listchglbl1.text = str(' ')
            
        try:
            list2 = str(istlist[2])[2:-2]
            urllist2 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list2,list2))
            souplist2 = bs4.BeautifulSoup(urllist2.text, features="html.parser")
            list2name = souplist2.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list2price = souplist2.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list2chgp = souplist2.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl2.text = str(list2chgp)
                except AttributeError:
                    list2chgn = souplist2.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl2.text = str(list2chgn)
            except AttributeError:
                list2chg = souplist2.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl2.text = str(list2chg)
            self.ids.listnamelbl2.text = str(list2name)
            self.ids.listlbl2.text = str(list2)
            self.ids.listpricelbl2.text = str(list2price)
        except (IndexError, AttributeError):
            self.ids.listlbl2.text = str(' ')
            self.ids.listnamelbl2.text = str(' ')
            self.ids.listpricelbl2.text = str(' ')
            self.ids.listchglbl2.text = str(' ')
            
        try:
            list3 = str(istlist[3])[2:-2]
            urllist3 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list3,list3))
            souplist3 = bs4.BeautifulSoup(urllist3.text, features="html.parser")
            list3name = souplist3.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list3price = souplist3.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list3chgp = souplist3.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl3.text = str(list3chgp)
                except AttributeError:
                    list3chgn = souplist3.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl3.text = str(list3chgn)
            except AttributeError:
                list3chg = souplist3.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl3.text = str(list3chg)
            self.ids.listnamelbl3.text = str(list3name)
            self.ids.listlbl3.text = str(istlist[3])[2:-2]
            self.ids.listpricelbl3.text = str(list3price)
        except (IndexError, AttributeError):
            self.ids.listlbl3.text = str(' ')
            self.ids.listnamelbl3.text = str(' ')
            self.ids.listpricelbl3.text = str(' ')
            self.ids.listchglbl3.text = str(' ')
            
        try:
            list4 = str(istlist[4])[2:-2]
            urllist4 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list4,list4))
            souplist4 = bs4.BeautifulSoup(urllist4.text, features="html.parser")
            list4name = souplist4.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list4price = souplist4.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list4chgp = souplist4.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl4.text = str(list4chgp)
                except AttributeError:
                    list4chgn = souplist4.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl4.text = str(list4chgn)
            except AttributeError:
                list4chg = souplist4.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl4.text = str(list4chg)
            self.ids.listnamelbl4.text = str(list4name)
            self.ids.listlbl4.text = str(istlist[4])[2:-2]
            self.ids.listpricelbl4.text = str(list4price)
        except (IndexError, AttributeError):
            self.ids.listlbl4.text = str(' ')
            self.ids.listnamelbl4.text = str(' ')
            self.ids.listpricelbl4.text = str(' ')
            self.ids.listchglbl4.text = str(' ')
            
        try:
            list5 = str(istlist[5])[2:-2]
            urllist5 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list5,list5))
            souplist5 = bs4.BeautifulSoup(urllist5.text, features="html.parser")
            list5name = souplist5.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list5price = souplist5.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list5chgp = souplist5.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl5.text = str(list5chgp)
                except AttributeError:
                    list5chgn = souplist5.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl5.text = str(list5chgn)
            except AttributeError:
                list5chg = souplist5.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl5.text = str(list5chg)
            self.ids.listnamelbl5.text = str(list5name)
            self.ids.listlbl5.text = str(istlist[5])[2:-2]
            self.ids.listpricelbl5.text = str(list5price)
        except (IndexError, AttributeError):
            self.ids.listlbl5.text = str(' ')
            self.ids.listnamelbl5.text = str(' ')
            self.ids.listpricelbl5.text = str(' ')
            self.ids.listchglbl5.text = str(' ')
            
        try:
            list6 = str(istlist[6])[2:-2]
            urllist6 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list6,list6))
            souplist6 = bs4.BeautifulSoup(urllist6.text, features="html.parser")
            list6name = souplist6.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list6price = souplist6.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list6chgp = souplist6.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl6.text = str(list6chgp)
                except AttributeError:
                    list6chgn = souplist6.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl6.text = str(list6chgn)
            except AttributeError:
                list6chg = souplist6.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl6.text = str(list6chg)
            self.ids.listnamelbl6.text = str(list6name)
            self.ids.listlbl6.text = str(istlist[6])[2:-2]
            self.ids.listpricelbl6.text = str(list6price)
        except (IndexError, AttributeError):
            self.ids.listlbl6.text = str(' ')
            self.ids.listnamelbl6.text = str(' ')
            self.ids.listpricelbl6.text = str(' ')
            self.ids.listchglbl6.text = str(' ')
            
        try:
            list7 = str(istlist[7])[2:-2]
            urllist7 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list7,list7))
            souplist7 = bs4.BeautifulSoup(urllist7.text, features="html.parser")
            list7name = souplist7.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list7price = souplist7.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list7chgp = souplist7.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl7.text = str(list7chgp)
                except AttributeError:
                    list7chgn = souplist7.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl7.text = str(list7chgn)
            except AttributeError:
                list7chg = souplist7.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl7.text = str(list7chg)
            self.ids.listnamelbl7.text = str(list7name)
            self.ids.listlbl7.text = str(istlist[7])[2:-2]
            self.ids.listpricelbl7.text = str(list7price)
        except (IndexError, AttributeError):
            self.ids.listlbl7.text = str(' ')
            self.ids.listnamelbl7.text = str(' ')
            self.ids.listpricelbl7.text = str(' ')
            self.ids.listchglbl7.text = str(' ')
            
        try:
            list8 = str(istlist[8])[2:-2]
            urllist8 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list8,list8))
            souplist8 = bs4.BeautifulSoup(urllist8.text, features="html.parser")
            list8name = souplist8.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list8price = souplist8.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list8chgp = souplist8.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl8.text = str(list8chgp)
                except AttributeError:
                    list8chgn = souplist8.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl8.text = str(list8chgn)
            except AttributeError:
                list8chg = souplist8.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl8.text = str(list8chg)
            self.ids.listnamelbl8.text = str(list8name)
            self.ids.listlbl8.text = str(istlist[8])[2:-2]
            self.ids.listpricelbl8.text = str(list8price)
        except (IndexError, AttributeError):
            self.ids.listlbl8.text = str(' ')
            self.ids.listnamelbl8.text = str(' ')
            self.ids.listpricelbl8.text = str(' ')
            self.ids.listchglbl8.text = str(' ')
            
        try:
            list9 = str(istlist[9])[2:-2]
            urllist9 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list9,list9))
            souplist9 = bs4.BeautifulSoup(urllist9.text, features="html.parser")
            list9name = souplist9.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list9price = souplist9.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list9chgp = souplist9.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl9.text = str(list9chgp)
                except AttributeError:
                    list9chgn = souplist9.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl9.text = str(list9chgn)
            except AttributeError:
                list9chg = souplist9.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl9.text = str(list9chg)
            self.ids.listnamelbl9.text = str(list9name)
            self.ids.listlbl9.text = str(istlist[9])[2:-2]
            self.ids.listpricelbl9.text = str(list9price)
        except (IndexError, AttributeError):
            self.ids.listlbl9.text = str(' ')
            self.ids.listnamelbl9.text = str(' ')
            self.ids.listpricelbl9.text = str(' ')
            self.ids.listchglbl9.text = str(' ')
            
        try:
            list10 = str(istlist[10])[2:-2]
            urllist10 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list10,list10))
            souplist10 = bs4.BeautifulSoup(urllist10.text, features="html.parser")
            list10name = souplist10.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list10price = souplist10.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list10chgp = souplist10.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl10.text = str(list10chgp)
                except AttributeError:
                    list10chgn = souplist10.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl10.text = str(list10chgn)
            except AttributeError:
                list10chg = souplist10.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl10.text = str(list10chg)
            self.ids.listnamelbl10.text = str(list10name)
            self.ids.listlbl10.text = str(istlist[10])[2:-2]
            self.ids.listpricelbl10.text = str(list10price)
        except (IndexError, AttributeError):
            self.ids.listlbl10.text = str(' ')
            self.ids.listnamelbl10.text = str(' ')
            self.ids.listpricelbl10.text = str(' ')
            self.ids.listchglbl10.text = str(' ')
            
        try:
            list11 = str(istlist[11])[2:-2]
            urllist11 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list11,list11))
            souplist11 = bs4.BeautifulSoup(urllist11.text, features="html.parser")
            list11name = souplist11.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list11price = souplist11.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list11chgp = souplist11.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl11.text = str(list11chgp)
                except AttributeError:
                    list11chgn = souplist11.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl11.text = str(list11chgn)
            except AttributeError:
                list11chg = souplist11.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl11.text = str(list11chg)
            self.ids.listnamelbl11.text = str(list11name)
            self.ids.listlbl11.text = str(istlist[11])[2:-2]
            self.ids.listpricelbl11.text = str(list11price)
        except (IndexError, AttributeError):
            self.ids.listlbl11.text = str(' ')
            self.ids.listnamelbl11.text = str(' ')
            self.ids.listpricelbl11.text = str(' ')
            self.ids.listchglbl11.text = str(' ')
            
        try:
            list12 = str(istlist[12])[2:-2]
            urllist12 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list12,list12))
            souplist12 = bs4.BeautifulSoup(urllist12.text, features="html.parser")
            list12name = souplist12.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list12price = souplist12.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list12chgp = souplist12.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl12.text = str(list12chgp)
                except AttributeError:
                    list12chgn = souplist12.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl12.text = str(list12chgn)
            except AttributeError:
                list12chg = souplist12.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl12.text = str(list12chg)
            self.ids.listnamelbl12.text = str(list12name)
            self.ids.listlbl12.text = str(istlist[12])[2:-2]
            self.ids.listpricelbl12.text = str(list12price)
        except (IndexError, AttributeError):
            self.ids.listlbl12.text = str(' ')
            self.ids.listnamelbl12.text = str(' ')
            self.ids.listpricelbl12.text = str(' ')
            self.ids.listchglbl12.text = str(' ')
            
        try:
            list13 = str(istlist[13])[2:-2]
            urllist13 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list13,list13))
            souplist13 = bs4.BeautifulSoup(urllist13.text, features="html.parser")
            list13name = souplist13.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list13price = souplist13.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list13chgp = souplist13.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl13.text = str(list13chgp)
                except AttributeError:
                    list13chgn = souplist13.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl13.text = str(list13chgn)
            except AttributeError:
                list13chg = souplist13.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl13.text = str(list13chg)
            self.ids.listnamelbl13.text = str(list13name)
            self.ids.listlbl13.text = str(istlist[13])[2:-2]
            self.ids.listpricelbl13.text = str(list13price)
        except (IndexError, AttributeError):
            self.ids.listlbl13.text = str(' ')
            self.ids.listnamelbl13.text = str(' ')
            self.ids.listpricelbl13.text = str(' ')
            self.ids.listchglbl13.text = str(' ')
            
        try:
            list14 = str(istlist[14])[2:-2]
            urllist14 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list14,list14))
            souplist14 = bs4.BeautifulSoup(urllist14.text, features="html.parser")
            list14name = souplist14.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list14price = souplist14.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list14chgp = souplist14.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl14.text = str(list14chgp)
                except AttributeError:
                    list14chgn = souplist14.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl14.text = str(list14chgn)
            except AttributeError:
                list14chg = souplist14.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl14.text = str(list14chg)
            self.ids.listnamelbl14.text = str(list14name)
            self.ids.listlbl14.text = str(istlist[14])[2:-2]
            self.ids.listpricelbl14.text = str(list14price)
        except (IndexError, AttributeError):
            self.ids.listlbl14.text = str(' ')
            self.ids.listnamelbl14.text = str(' ')
            self.ids.listpricelbl14.text = str(' ')
            self.ids.listchglbl14.text = str(' ')
            
        try:
            list15 = str(istlist[15])[2:-2]
            urllist15 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list15,list15))
            souplist15 = bs4.BeautifulSoup(urllist15.text, features="html.parser")
            list15name = souplist15.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list15price = souplist15.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list15chgp = souplist15.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl15.text = str(list15chgp)
                except AttributeError:
                    list15chgn = souplist15.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl15.text = str(list15chgn)
            except AttributeError:
                list15chg = souplist15.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl15.text = str(list15chg)
            self.ids.listnamelbl15.text = str(list15name)
            self.ids.listlbl15.text = str(istlist[15])[2:-2]
            self.ids.listpricelbl15.text = str(list15price)
        except (IndexError, AttributeError):
            self.ids.listlbl15.text = str(' ')
            self.ids.listnamelbl15.text = str(' ')
            self.ids.listpricelbl15.text = str(' ')
            self.ids.listchglbl15.text = str(' ')
            
        try:
            list16 = str(istlist[16])[2:-2]
            urllist16 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list16,list16))
            souplist16 = bs4.BeautifulSoup(urllist16.text, features="html.parser")
            list16name = souplist16.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list16price = souplist16.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list16chgp = souplist16.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl16.text = str(list16chgp)
                except AttributeError:
                    list16chgn = souplist16.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl16.text = str(list16chgn)
            except AttributeError:
                list16chg = souplist16.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl16.text = str(list16chg)
            self.ids.listnamelbl16.text = str(list16name)
            self.ids.listlbl16.text = str(istlist[16])[2:-2]
            self.ids.listpricelbl16.text = str(list16price)
        except (IndexError, AttributeError):
            self.ids.listlbl16.text = str(' ')
            self.ids.listnamelbl16.text = str(' ')
            self.ids.listpricelbl16.text = str(' ')
            self.ids.listchglbl16.text = str(' ')
            
        try:
            list17 = str(istlist[17])[2:-2]
            urllist17 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list17,list17))
            souplist17 = bs4.BeautifulSoup(urllist17.text, features="html.parser")
            list17name = souplist17.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list17price = souplist17.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list17chgp = souplist17.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl17.text = str(list17chgp)
                except AttributeError:
                    list17chgn = souplist17.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl17.text = str(list17chgn)
            except AttributeError:
                list17chg = souplist17.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl17.text = str(list17chg)
            self.ids.listnamelbl17.text = str(list17name)
            self.ids.listlbl17.text = str(istlist[17])[2:-2]
            self.ids.listpricelbl17.text = str(list17price)
        except (IndexError, AttributeError):
            self.ids.listlbl17.text = str(' ')
            self.ids.listnamelbl17.text = str(' ')
            self.ids.listpricelbl17.text = str(' ')
            self.ids.listchglbl17.text = str(' ')
            
        try:
            list18 = str(istlist[18])[2:-2]
            urllist18 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list18,list18))
            souplist18 = bs4.BeautifulSoup(urllist18.text, features="html.parser")
            list18name = souplist18.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list18price = souplist18.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list18chgp = souplist18.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl18.text = str(list18chgp)
                except AttributeError:
                    list18chgn = souplist18.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl18.text = str(list18chgn)
            except AttributeError:
                list18chg = souplist18.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl18.text = str(list18chg)
            self.ids.listnamelbl18.text = str(list18name)
            self.ids.listlbl18.text = str(istlist[18])[2:-2]
            self.ids.listpricelbl18.text = str(list18price)
        except (IndexError, AttributeError):
            self.ids.listlbl18.text = str(' ')
            self.ids.listnamelbl18.text = str(' ')
            self.ids.listpricelbl18.text = str(' ')
            self.ids.listchglbl18.text = str(' ')
            
        try:
            list19 = str(istlist[19])[2:-2]
            urllist19 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list19,list19))
            souplist19 = bs4.BeautifulSoup(urllist19.text, features="html.parser")
            list19name = souplist19.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list19price = souplist19.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list19chgp = souplist19.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl19.text = str(list19chgp)
                except AttributeError:
                    list19chgn = souplist19.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl19.text = str(list19chgn)
            except AttributeError:
                list19chg = souplist19.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl19.text = str(list19chg)
            self.ids.listnamelbl19.text = str(list19name)
            self.ids.listlbl19.text = str(istlist[19])[2:-2]
            self.ids.listpricelbl19.text = str(list19price)
        except (IndexError, AttributeError):
            self.ids.listlbl19.text = str(' ')
            self.ids.listnamelbl19.text = str(' ')
            self.ids.listpricelbl19.text = str(' ')
            self.ids.listchglbl19.text = str(' ')
            
        try:
            list20 = str(istlist[20])[2:-2]
            urllist20 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list20,list20))
            souplist20 = bs4.BeautifulSoup(urllist20.text, features="html.parser")
            list20name = souplist20.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list20price = souplist20.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list20chgp = souplist20.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl20.text = str(list20chgp)
                except AttributeError:
                    list20chgn = souplist20.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl20.text = str(list20chgn)
            except AttributeError:
                list20chg = souplist20.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl20.text = str(list20chg)
            self.ids.listnamelbl20.text = str(list20name)
            self.ids.listlbl20.text = str(istlist[20])[2:-2]
            self.ids.listpricelbl20.text = str(list20price)
        except (IndexError, AttributeError):
            self.ids.listlbl20.text = str(' ')
            self.ids.listnamelbl20.text = str(' ')
            self.ids.listpricelbl20.text = str(' ')
            self.ids.listchglbl20.text = str(' ')
            
        try:
            list21 = str(istlist[21])[2:-2]
            urllist21 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list21,list21))
            souplist21 = bs4.BeautifulSoup(urllist21.text, features="html.parser")
            list21name = souplist21.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list21price = souplist21.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list21chgp = souplist21.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl21.text = str(list21chgp)
                except AttributeError:
                    list21chgn = souplist21.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl21.text = str(list21chgn)
            except AttributeError:
                list21chg = souplist21.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl21.text = str(list21chg)
            self.ids.listnamelbl21.text = str(list21name)
            self.ids.listlbl21.text = str(istlist[21])[2:-2]
            self.ids.listpricelbl21.text = str(list21price)
        except (IndexError, AttributeError):
            self.ids.listlbl21.text = str(' ')
            self.ids.listnamelbl21.text = str(' ')
            self.ids.listpricelbl21.text = str(' ')
            self.ids.listchglbl21.text = str(' ')
            
        try:
            list22 = str(istlist[22])[2:-2]
            urllist22 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list22,list22))
            souplist22 = bs4.BeautifulSoup(urllist22.text, features="html.parser")
            list22name = souplist22.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list22price = souplist22.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list22chgp = souplist22.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl22.text = str(list22chgp)
                except AttributeError:
                    list22chgn = souplist22.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl22.text = str(list22chgn)
            except AttributeError:
                list22chg = souplist22.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl22.text = str(list22chg)
            self.ids.listnamelbl22.text = str(list22name)
            self.ids.listlbl22.text = str(istlist[22])[2:-2]
            self.ids.listpricelbl22.text = str(list22price)
        except (IndexError, AttributeError):
            self.ids.listlbl22.text = str(' ')
            self.ids.listnamelbl22.text = str(' ')
            self.ids.listpricelbl22.text = str(' ')
            self.ids.listchglbl22.text = str(' ')
            
        try:
            list23 = str(istlist[23])[2:-2]
            urllist23 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list23,list23))
            souplist23 = bs4.BeautifulSoup(urllist23.text, features="html.parser")
            list23name = souplist23.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list23price = souplist23.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list23chgp = souplist23.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl23.text = str(list23chgp)
                except AttributeError:
                    list23chgn = souplist23.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl23.text = str(list23chgn)
            except AttributeError:
                list23chg = souplist23.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl23.text = str(list23chg)
            self.ids.listnamelbl23.text = str(list23name)
            self.ids.listlbl23.text = str(istlist[23])[2:-2]
            self.ids.listpricelbl23.text = str(list23price)
        except (IndexError, AttributeError):
            self.ids.listlbl23.text = str(' ')
            self.ids.listnamelbl23.text = str(' ')
            self.ids.listpricelbl23.text = str(' ')
            self.ids.listchglbl23.text = str(' ')
            
        try:
            list24 = str(istlist[24])[2:-2]
            urllist24 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list24,list24))
            souplist24 = bs4.BeautifulSoup(urllist24.text, features="html.parser")
            list24name = souplist24.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list24price = souplist24.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list24chgp = souplist24.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl24.text = str(list24chgp)
                except AttributeError:
                    list24chgn = souplist24.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl24.text = str(list24chgn)
            except AttributeError:
                list24chg = souplist24.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl24.text = str(list24chg)
            self.ids.listnamelbl24.text = str(list24name)
            self.ids.listlbl24.text = str(istlist[24])[2:-2]
            self.ids.listpricelbl24.text = str(list24price)
        except (IndexError, AttributeError):
            self.ids.listlbl24.text = str(' ')
            self.ids.listnamelbl24.text = str(' ')
            self.ids.listpricelbl24.text = str(' ')
            self.ids.listchglbl24.text = str(' ')
            
        try:
            list25 = str(istlist[25])[2:-2]
            urllist25 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list25,list25))
            souplist25 = bs4.BeautifulSoup(urllist25.text, features="html.parser")
            list25name = souplist25.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list25price = souplist25.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list25chgp = souplist25.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl25.text = str(list25chgp)
                except AttributeError:
                    list25chgn = souplist25.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl25.text = str(list25chgn)
            except AttributeError:
                list25chg = souplist25.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl25.text = str(list25chg)
            self.ids.listnamelbl25.text = str(list25name)
            self.ids.listlbl25.text = str(istlist[25])[2:-2]
            self.ids.listpricelbl25.text = str(list25price)
        except (IndexError, AttributeError):
            self.ids.listlbl25.text = str(' ')
            self.ids.listnamelbl25.text = str(' ')
            self.ids.listpricelbl25.text = str(' ')
            self.ids.listchglbl25.text = str(' ')
            
        try:
            list26 = str(istlist[26])[2:-2]
            urllist26 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list26,list26))
            souplist26 = bs4.BeautifulSoup(urllist26.text, features="html.parser")
            list26name = souplist26.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list26price = souplist26.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list26chgp = souplist26.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl26.text = str(list26chgp)
                except AttributeError:
                    list26chgn = souplist26.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl26.text = str(list26chgn)
            except AttributeError:
                list26chg = souplist26.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl26.text = str(list26chg)
            self.ids.listnamelbl26.text = str(list26name)
            self.ids.listlbl26.text = str(istlist[26])[2:-2]
            self.ids.listpricelbl26.text = str(list26price)
        except (IndexError, AttributeError):
            self.ids.listlbl26.text = str(' ')
            self.ids.listnamelbl26.text = str(' ')
            self.ids.listpricelbl26.text = str(' ')
            self.ids.listchglbl26.text = str(' ')
            
        try:
            list27 = str(istlist[27])[2:-2]
            urllist27 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list27,list27))
            souplist27 = bs4.BeautifulSoup(urllist27.text, features="html.parser")
            list27name = souplist27.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list27price = souplist27.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list27chgp = souplist27.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl27.text = str(list27chgp)
                except AttributeError:
                    list27chgn = souplist27.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl27.text = str(list27chgn)
            except AttributeError:
                list27chg = souplist27.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl27.text = str(list27chg)
            self.ids.listnamelbl27.text = str(list27name)
            self.ids.listlbl27.text = str(istlist[27])[2:-2]
            self.ids.listpricelbl27.text = str(list27price)
        except (IndexError, AttributeError):
            self.ids.listlbl27.text = str(' ')
            self.ids.listnamelbl27.text = str(' ')
            self.ids.listpricelbl27.text = str(' ')
            self.ids.listchglbl27.text = str(' ')
            
        try:
            list28 = str(istlist[28])[2:-2]
            urllist28 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list28,list28))
            souplist28 = bs4.BeautifulSoup(urllist28.text, features="html.parser")
            list28name = souplist28.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list28price = souplist28.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list28chgp = souplist28.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl28.text = str(list28chgp)
                except AttributeError:
                    list28chgn = souplist28.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl28.text = str(list28chgn)
            except AttributeError:
                list28chg = souplist28.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl28.text = str(list28chg)
            self.ids.listnamelbl28.text = str(list28name)
            self.ids.listlbl28.text = str(istlist[28])[2:-2]
            self.ids.listpricelbl28.text = str(list28price)
        except (IndexError, AttributeError):
            self.ids.listlbl28.text = str(' ')
            self.ids.listnamelbl28.text = str(' ')
            self.ids.listpricelbl28.text = str(' ')
            self.ids.listchglbl28.text = str(' ')
            
        try:
            list29 = str(istlist[29])[2:-2]
            urllist29 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list29,list29))
            souplist29 = bs4.BeautifulSoup(urllist29.text, features="html.parser")
            list29name = souplist29.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list29price = souplist29.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list29chgp = souplist29.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl29.text = str(list29chgp)
                except AttributeError:
                    list29chgn = souplist29.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl29.text = str(list29chgn)
            except AttributeError:
                list29chg = souplist29.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl29.text = str(list29chg)
            self.ids.listnamelbl29.text = str(list29name)
            self.ids.listlbl29.text = str(istlist[29])[2:-2]
            self.ids.listpricelbl29.text = str(list29price)
        except (IndexError, AttributeError):
            self.ids.listlbl29.text = str(' ')
            self.ids.listnamelbl29.text = str(' ')
            self.ids.listpricelbl29.text = str(' ')
            self.ids.listchglbl29.text = str(' ')
            
        try:
            list30 = str(istlist[30])[2:-2]
            urllist30 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list30,list30))
            souplist30 = bs4.BeautifulSoup(urllist30.text, features="html.parser")
            list30name = souplist30.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list30price = souplist30.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list30chgp = souplist30.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl30.text = str(list30chgp)
                except AttributeError:
                    list30chgn = souplist30.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl30.text = str(list30chgn)
            except AttributeError:
                list30chg = souplist30.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl30.text = str(list30chg)
            self.ids.listnamelbl30.text = str(list30name)
            self.ids.listlbl30.text = str(istlist[30])[2:-2]
            self.ids.listpricelbl30.text = str(list30price)
        except (IndexError, AttributeError):
            self.ids.listlbl30.text = str(' ')
            self.ids.listnamelbl30.text = str(' ')
            self.ids.listpricelbl30.text = str(' ')
            self.ids.listchglbl30.text = str(' ')
            
        try:
            list31 = str(istlist[31])[2:-2]
            urllist31 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list31,list31))
            souplist31 = bs4.BeautifulSoup(urllist31.text, features="html.parser")
            list31name = souplist31.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list31price = souplist31.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list31chgp = souplist31.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl31.text = str(list31chgp)
                except AttributeError:
                    list31chgn = souplist31.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl31.text = str(list31chgn)
            except AttributeError:
                list31chg = souplist31.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl31.text = str(list31chg)
            self.ids.listnamelbl31.text = str(list31name)
            self.ids.listlbl31.text = str(istlist[31])[2:-2]
            self.ids.listpricelbl31.text = str(list31price)
        except (IndexError, AttributeError):
            self.ids.listlbl31.text = str(' ')
            self.ids.listnamelbl31.text = str(' ')
            self.ids.listpricelbl31.text = str(' ')
            self.ids.listchglbl31.text = str(' ')
            
        try:
            list32 = str(istlist[32])[2:-2]
            urllist32 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list32,list32))
            souplist32 = bs4.BeautifulSoup(urllist32.text, features="html.parser")
            list32name = souplist32.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list32price = souplist32.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list32chgp = souplist32.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl32.text = str(list32chgp)
                except AttributeError:
                    list32chgn = souplist32.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl32.text = str(list32chgn)
            except AttributeError:
                list32chg = souplist32.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl32.text = str(list32chg)
            self.ids.listnamelbl32.text = str(list32name)
            self.ids.listlbl32.text = str(istlist[32])[2:-2]
            self.ids.listpricelbl32.text = str(list32price)
        except (IndexError, AttributeError):
            self.ids.listlbl32.text = str(' ')
            self.ids.listnamelbl32.text = str(' ')
            self.ids.listpricelbl32.text = str(' ')
            self.ids.listchglbl32.text = str(' ')
            
        try:
            list33 = str(istlist[33])[2:-2]
            urllist33 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list33,list33))
            souplist33 = bs4.BeautifulSoup(urllist33.text, features="html.parser")
            list33name = souplist33.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list33price = souplist33.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list33chgp = souplist33.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl33.text = str(list33chgp)
                except AttributeError:
                    list33chgn = souplist33.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl33.text = str(list33chgn)
            except AttributeError:
                list33chg = souplist33.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl33.text = str(list33chg)
            self.ids.listnamelbl33.text = str(list33name)
            self.ids.listlbl33.text = str(istlist[33])[2:-2]
            self.ids.listpricelbl33.text = str(list33price)
        except (IndexError, AttributeError):
            self.ids.listlbl33.text = str(' ')
            self.ids.listnamelbl33.text = str(' ')
            self.ids.listpricelbl33.text = str(' ')
            self.ids.listchglbl33.text = str(' ')
            
        try:
            list34 = str(istlist[34])[2:-2]
            urllist34 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list34,list34))
            souplist34 = bs4.BeautifulSoup(urllist34.text, features="html.parser")
            list34name = souplist34.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list34price = souplist34.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list34chgp = souplist34.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl34.text = str(list34chgp)
                except AttributeError:
                    list34chgn = souplist34.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl34.text = str(list34chgn)
            except AttributeError:
                list34chg = souplist34.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl34.text = str(list34chg)
            self.ids.listnamelbl34.text = str(list34name)
            self.ids.listlbl34.text = str(istlist[34])[2:-2]
            self.ids.listpricelbl34.text = str(list34price)
        except (IndexError, AttributeError):
            self.ids.listlbl34.text = str(' ')
            self.ids.listnamelbl34.text = str(' ')
            self.ids.listpricelbl34.text = str(' ')
            self.ids.listchglbl34.text = str(' ')
            
        try:
            list35 = str(istlist[35])[2:-2]
            urllist35 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list35,list35))
            souplist35 = bs4.BeautifulSoup(urllist35.text, features="html.parser")
            list35name = souplist35.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list35price = souplist35.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list35chgp = souplist35.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl35.text = str(list35chgp)
                except AttributeError:
                    list35chgn = souplist35.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl35.text = str(list35chgn)
            except AttributeError:
                list35chg = souplist35.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl35.text = str(list35chg)
            self.ids.listnamelbl35.text = str(list35name)
            self.ids.listlbl35.text = str(istlist[35])[2:-2]
            self.ids.listpricelbl35.text = str(list35price)
        except (IndexError, AttributeError):
            self.ids.listlbl35.text = str(' ')
            self.ids.listnamelbl35.text = str(' ')
            self.ids.listpricelbl35.text = str(' ')
            self.ids.listchglbl35.text = str(' ')
            
        try:
            list36 = str(istlist[36])[2:-2]
            urllist36 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list36,list36))
            souplist36 = bs4.BeautifulSoup(urllist36.text, features="html.parser")
            list36name = souplist36.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list36price = souplist36.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list36chgp = souplist36.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl36.text = str(list36chgp)
                except AttributeError:
                    list36chgn = souplist36.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl36.text = str(list36chgn)
            except AttributeError:
                list36chg = souplist36.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl36.text = str(list36chg)
            self.ids.listnamelbl36.text = str(list36name)
            self.ids.listlbl36.text = str(istlist[36])[2:-2]
            self.ids.listpricelbl36.text = str(list36price)
        except (IndexError, AttributeError):
            self.ids.listlbl36.text = str(' ')
            self.ids.listnamelbl36.text = str(' ')
            self.ids.listpricelbl36.text = str(' ')
            self.ids.listchglbl36.text = str(' ')
            
        try:
            list37 = str(istlist[37])[2:-2]
            urllist37 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list37,list37))
            souplist37 = bs4.BeautifulSoup(urllist37.text, features="html.parser")
            list37name = souplist37.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list37price = souplist37.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list37chgp = souplist37.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl37.text = str(list37chgp)
                except AttributeError:
                    list37chgn = souplist37.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl37.text = str(list37chgn)
            except AttributeError:
                list37chg = souplist37.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl37.text = str(list37chg)
            self.ids.listnamelbl1.text = str(list37name)
            self.ids.listlbl37.text = str(istlist[37])[2:-2]
            self.ids.listpricelbl37.text = str(list37price)
        except (IndexError, AttributeError):
            self.ids.listlbl37.text = str(' ')
            self.ids.listnamelbl37.text = str(' ')
            self.ids.listpricelbl37.text = str(' ')
            self.ids.listchglbl37.text = str(' ')
            
        try:
            list38 = str(istlist[38])[2:-2]
            urllist38 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list38,list38))
            souplist38 = bs4.BeautifulSoup(urllist38.text, features="html.parser")
            list38name = souplist38.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list38price = souplist38.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list38chgp = souplist38.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl38.text = str(list38chgp)
                except AttributeError:
                    list38chgn = souplist38.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl38.text = str(list38chgn)
            except AttributeError:
                list38chg = souplist38.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl38.text = str(list38chg)
            self.ids.listnamelbl38.text = str(list38name)
            self.ids.listlbl38.text = str(istlist[38])[2:-2]
            self.ids.listpricelbl38.text = str(list38price)
        except (IndexError, AttributeError):
            self.ids.listlbl38.text = str(' ')
            self.ids.listnamelbl38.text = str(' ')
            self.ids.listpricelbl38.text = str(' ')
            self.ids.listchglbl38.text = str(' ')
            
        try:
            list39 = str(istlist[39])[2:-2]
            urllist39 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list39,list39))
            souplist39 = bs4.BeautifulSoup(urllist39.text, features="html.parser")
            list39name = souplist39.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list39price = souplist39.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list39chgp = souplist39.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl39.text = str(list39chgp)
                except AttributeError:
                    list39chgn = souplist39.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl39.text = str(list39chgn)
            except AttributeError:
                list39chg = souplist39.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl39.text = str(list39chg)
            self.ids.listnamelbl39.text = str(list39name)
            self.ids.listlbl39.text = str(istlist[39])[2:-2]
            self.ids.listpricelbl39.text = str(list39price)
        except (IndexError, AttributeError):
            self.ids.listlbl39.text = str(' ')
            self.ids.listnamelbl39.text = str(' ')
            self.ids.listpricelbl39.text = str(' ')
            self.ids.listchglbl39.text = str(' ')
            
        try:
            list40 = str(istlist[40])[2:-2]
            urllist40 = requests.get(('https://finance.yahoo.com/quote/%s?p=%s') % (list40,list40))
            souplist40 = bs4.BeautifulSoup(urllist40.text, features="html.parser")
            list40name = souplist40.find('h1',{'class': 'D(ib) Fz(18px)'}).text
            list40price = souplist40.find_all("div",{'class': 'My(6px) Pos(r) smartphone_Mt(6px)'})[0].find('span').text
            try:
                try:
                    list40chgp = souplist40.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($positiveColor)'}).text
                    self.ids.listchglbl40.text = str(list40chgp)
                except AttributeError:
                    list40chgn = souplist40.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px) C($negativeColor)'}).text
                    self.ids.listchglbl40.text = str(list40chgn)
            except AttributeError:
                list40chg = souplist40.find('span',{'class': 'Trsdu(0.3s) Fw(500) Pstart(10px) Fz(24px)'}).text
                self.ids.listchglbl40.text = str(list40chg)
            self.ids.listnamelbl40.text = str(list40name)
            self.ids.listlbl40.text = str(istlist[40])[2:-2]
            self.ids.listpricelbl40.text = str(list40price)
        except (IndexError, AttributeError):
            self.ids.listlbl40.text = str(' ')
            self.ids.listnamelbl40.text = str(' ')
            self.ids.listpricelbl40.text = str(' ')
            self.ids.listchglbl40.text = str(' ')
        slistfile.close()
        

    def delist1(self):
        slistfiledel1 = open('Stocklist.csv')
        slistdel1 = pd.read_csv('stocklist.csv',index_col=0)
        slist1 = csv.reader(slistfiledel1)
        istlist1 = list(slist1)
        deltick = str(istlist1[1])[2:-2]
        slistdel1.drop(deltick)
        #del slistdel1[str(deltick)]
        slistfiledel1.close()
        

    def addlist(self):
        newtick = self.ids.inputlist.text
        newtickcap = newtick.upper()
        newlisttick = [newtickcap]
        with open('stocklist.csv', 'a', newline='') as w:
            writelist = csv.writer(w)
            writelist.writerow(newlisttick)
        self.listref()


class curanscreen(Screen):
    pass

    def gainref(self):

            urlgainers = requests.get('https://www.tradingview.com/markets/stocks-usa/market-movers-gainers/')
            soupgainerstick = bs4.BeautifulSoup(urlgainers.text, features="html.parser")

            gainerhtml0 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[0]
            gainerhtml1 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[1]
            gainerhtml2 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[2]
            gainerhtml3 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[3]
            gainerhtml4 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[4]
            gainerhtml5 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[5]
            gainerhtml6 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[6]
            gainerhtml7 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[7]
            gainerhtml8 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[8]
            gainerhtml9 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[9]
            gainerhtml10 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[10]
            gainerhtml11 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[11]
            gainerhtml12 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[12]
            gainerhtml13 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[13]
            gainerhtml14 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[14]
            gainerhtml15 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[15]
            gainerhtml16 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[16]
            gainerhtml17 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[17]
            gainerhtml18 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[18]
            gainerhtml19 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[19]
            gainerhtml20 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[20]
            gainerhtml21 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[21]
            gainerhtml22 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[22]
            gainerhtml23 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[23]
            gainerhtml24 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[24]
            gainerhtml25 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[25]
            gainerhtml26 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[26]
            gainerhtml27 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[27]
            gainerhtml28 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[28]
            gainerhtml29 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[29]
            gainerhtml30 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[30]
            gainerhtml31 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[31]
            gainerhtml32 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[32]
            gainerhtml33 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[33]
            gainerhtml34 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[34]
            gainerhtml35 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[35]
            gainerhtml36 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[36]
            gainerhtml37 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[37]
            gainerhtml38 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[38]
            gainerhtml39 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[39]
            gainerhtml40 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[40]
            gainerhtml41 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[41]
            gainerhtml42 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[42]
            gainerhtml43 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[43]
            gainerhtml44 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[44]
            gainerhtml45 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[45]
            gainerhtml46 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[46]
            gainerhtml47 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[47]
            gainerhtml48 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[48]
            gainerhtml49 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[49]

            gainerstick0 = gainerhtml0.find('a').text
            gainerstick1 = gainerhtml1.find('a').text
            gainerstick2 = gainerhtml2.find('a').text
            gainerstick3 = gainerhtml3.find('a').text
            gainerstick4 = gainerhtml4.find('a').text
            gainerstick5 = gainerhtml5.find('a').text
            gainerstick6 = gainerhtml6.find('a').text
            gainerstick7 = gainerhtml7.find('a').text
            gainerstick8 = gainerhtml8.find('a').text
            gainerstick9 = gainerhtml9.find('a').text
            gainerstick10 = gainerhtml10.find('a').text
            gainerstick11 = gainerhtml11.find('a').text
            gainerstick12 = gainerhtml12.find('a').text
            gainerstick13 = gainerhtml13.find('a').text
            gainerstick14 = gainerhtml14.find('a').text
            gainerstick15 = gainerhtml15.find('a').text
            gainerstick16 = gainerhtml16.find('a').text
            gainerstick17 = gainerhtml17.find('a').text
            gainerstick18 = gainerhtml18.find('a').text
            gainerstick19 = gainerhtml19.find('a').text
            gainerstick20 = gainerhtml20.find('a').text
            gainerstick21 = gainerhtml21.find('a').text
            gainerstick22 = gainerhtml22.find('a').text
            gainerstick23 = gainerhtml23.find('a').text
            gainerstick24 = gainerhtml24.find('a').text
            gainerstick25 = gainerhtml25.find('a').text
            gainerstick26 = gainerhtml26.find('a').text
            gainerstick27 = gainerhtml27.find('a').text
            gainerstick28 = gainerhtml28.find('a').text
            gainerstick29 = gainerhtml29.find('a').text
            gainerstick30 = gainerhtml30.find('a').text
            gainerstick31 = gainerhtml31.find('a').text
            gainerstick32 = gainerhtml32.find('a').text
            gainerstick33 = gainerhtml33.find('a').text
            gainerstick34 = gainerhtml34.find('a').text
            gainerstick35 = gainerhtml35.find('a').text
            gainerstick36 = gainerhtml36.find('a').text
            gainerstick37 = gainerhtml37.find('a').text
            gainerstick38 = gainerhtml38.find('a').text
            gainerstick39 = gainerhtml39.find('a').text
            gainerstick40 = gainerhtml40.find('a').text
            gainerstick41 = gainerhtml41.find('a').text
            gainerstick42 = gainerhtml42.find('a').text
            gainerstick43 = gainerhtml43.find('a').text
            gainerstick44 = gainerhtml44.find('a').text
            gainerstick45 = gainerhtml45.find('a').text
            gainerstick46 = gainerhtml46.find('a').text
            gainerstick47 = gainerhtml47.find('a').text
            gainerstick48 = gainerhtml48.find('a').text
            gainerstick49 = gainerhtml49.find('a').text
            
            self.ids.gainlbl0.text = gainerstick0
            self.ids.gainlbl1.text = gainerstick1
            self.ids.gainlbl2.text = gainerstick2
            self.ids.gainlbl3.text = gainerstick3
            self.ids.gainlbl4.text = gainerstick4
            self.ids.gainlbl5.text = gainerstick5
            self.ids.gainlbl6.text = gainerstick6
            self.ids.gainlbl7.text = gainerstick7
            self.ids.gainlbl8.text = gainerstick8
            self.ids.gainlbl9.text = gainerstick9
            self.ids.gainlbl10.text = gainerstick10
            self.ids.gainlbl11.text = gainerstick11
            self.ids.gainlbl12.text = gainerstick12
            self.ids.gainlbl13.text = gainerstick13
            self.ids.gainlbl14.text = gainerstick14
            self.ids.gainlbl15.text = gainerstick15
            self.ids.gainlbl16.text = gainerstick16
            self.ids.gainlbl17.text = gainerstick17
            self.ids.gainlbl18.text = gainerstick18
            self.ids.gainlbl19.text = gainerstick19
            self.ids.gainlbl20.text = gainerstick20
            self.ids.gainlbl21.text = gainerstick21
            self.ids.gainlbl22.text = gainerstick22
            self.ids.gainlbl23.text = gainerstick23
            self.ids.gainlbl24.text = gainerstick24
            self.ids.gainlbl25.text = gainerstick25
            self.ids.gainlbl26.text = gainerstick26
            self.ids.gainlbl27.text = gainerstick27
            self.ids.gainlbl28.text = gainerstick28
            self.ids.gainlbl29.text = gainerstick29
            self.ids.gainlbl30.text = gainerstick30
            self.ids.gainlbl31.text = gainerstick31
            self.ids.gainlbl32.text = gainerstick32
            self.ids.gainlbl33.text = gainerstick33
            self.ids.gainlbl34.text = gainerstick34
            self.ids.gainlbl35.text = gainerstick35
            self.ids.gainlbl36.text = gainerstick36
            self.ids.gainlbl37.text = gainerstick37
            self.ids.gainlbl38.text = gainerstick38
            self.ids.gainlbl39.text = gainerstick39
            self.ids.gainlbl40.text = gainerstick40
            self.ids.gainlbl41.text = gainerstick41
            self.ids.gainlbl42.text = gainerstick42
            self.ids.gainlbl43.text = gainerstick43
            self.ids.gainlbl44.text = gainerstick44
            self.ids.gainlbl45.text = gainerstick45
            self.ids.gainlbl46.text = gainerstick46
            self.ids.gainlbl47.text = gainerstick47
            self.ids.gainlbl48.text = gainerstick48
            self.ids.gainlbl49.text = gainerstick49

            listrating0 = []
            for td in gainerhtml0:
                grate0 = td.find('span')
                listrating0.append(grate0)
            lr0 = pd.DataFrame(listrating0)
            grating0 = lr0.iloc[6]
            self.ids.gainlblrt0.text = grating0.to_string()[1:]

            listrating1 = []
            for td in gainerhtml1:
                grate1 = td.find('span')
                listrating1.append(grate1)
            lr1 = pd.DataFrame(listrating1)
            grating1 = lr1.iloc[6]
            self.ids.gainlblrt1.text = grating1.to_string()[1:]

            listrating2 = []
            for td in gainerhtml2:
                grate2 = td.find('span')
                listrating2.append(grate2)
            lr2 = pd.DataFrame(listrating2)
            grating2 = lr2.iloc[6]
            self.ids.gainlblrt2.text = grating2.to_string()[1:]

            listrating3 = []
            for td in gainerhtml3:
                grate3 = td.find('span')
                listrating3.append(grate3)
            lr3 = pd.DataFrame(listrating3)
            grating3 = lr3.iloc[6]
            self.ids.gainlblrt3.text = grating3.to_string()[1:]

            listrating4 = []
            for td in gainerhtml4:
                grate4 = td.find('span')
                listrating4.append(grate4)
            lr4 = pd.DataFrame(listrating4)
            grating4 = lr4.iloc[6]
            self.ids.gainlblrt4.text = grating4.to_string()[1:]

            listrating5 = []
            for td in gainerhtml5:
                grate5 = td.find('span')
                listrating5.append(grate5)
            lr5 = pd.DataFrame(listrating5)
            grating5 = lr5.iloc[6]
            self.ids.gainlblrt5.text = grating5.to_string()[1:]

            listrating6 = []
            for td in gainerhtml6:
                grate6 = td.find('span')
                listrating6.append(grate6)
            lr6 = pd.DataFrame(listrating6)
            grating6 = lr6.iloc[6]
            self.ids.gainlblrt6.text = grating6.to_string()[1:]

            listrating7 = []
            for td in gainerhtml7:
                grate7 = td.find('span')
                listrating7.append(grate7)
            lr7 = pd.DataFrame(listrating7)
            grating7 = lr7.iloc[6]
            self.ids.gainlblrt7.text = grating7.to_string()[1:]

            listrating8 = []
            for td in gainerhtml8:
                grate8 = td.find('span')
                listrating8.append(grate8)
            lr8 = pd.DataFrame(listrating8)
            grating8 = lr8.iloc[6]
            self.ids.gainlblrt8.text = grating8.to_string()[1:]

            listrating9 = []
            for td in gainerhtml9:
                grate9 = td.find('span')
                listrating9.append(grate9)
            lr9 = pd.DataFrame(listrating9)
            grating9 = lr9.iloc[6]
            self.ids.gainlblrt9.text = grating9.to_string()[1:]

            listrating10 = []
            for td in gainerhtml10:
                grate10 = td.find('span')
                listrating10.append(grate10)
            lr10 = pd.DataFrame(listrating10)
            grating10 = lr10.iloc[6]
            self.ids.gainlblrt10.text = grating10.to_string()[1:]

            listrating11 = []
            for td in gainerhtml11:
                grate11 = td.find('span')
                listrating11.append(grate11)
            lr11 = pd.DataFrame(listrating11)
            grating11 = lr11.iloc[6]
            self.ids.gainlblrt11.text = grating11.to_string()[1:]

            listrating12 = []
            for td in gainerhtml12:
                grate12 = td.find('span')
                listrating12.append(grate12)
            lr12 = pd.DataFrame(listrating12)
            grating12 = lr12.iloc[6]
            self.ids.gainlblrt12.text = grating12.to_string()[1:]

            listrating13 = []
            for td in gainerhtml13:
                grate13 = td.find('span')
                listrating13.append(grate13)
            lr13 = pd.DataFrame(listrating13)
            grating13 = lr13.iloc[6]
            self.ids.gainlblrt13.text = grating13.to_string()[1:]

            listrating14 = []
            for td in gainerhtml14:
                grate14 = td.find('span')
                listrating14.append(grate14)
            lr14 = pd.DataFrame(listrating14)
            grating14 = lr14.iloc[6]
            self.ids.gainlblrt14.text = grating14.to_string()[1:]

            listrating15 = []
            for td in gainerhtml15:
                grate15 = td.find('span')
                listrating15.append(grate15)
            lr15 = pd.DataFrame(listrating15)
            grating15 = lr15.iloc[6]
            self.ids.gainlblrt15.text = grating15.to_string()[1:]

            listrating16 = []
            for td in gainerhtml16:
                grate16 = td.find('span')
                listrating16.append(grate16)
            lr16 = pd.DataFrame(listrating16)
            grating16 = lr16.iloc[6]
            self.ids.gainlblrt16.text = grating16.to_string()[1:]

            listrating17 = []
            for td in gainerhtml17:
                grate17 = td.find('span')
                listrating17.append(grate17)
            lr17 = pd.DataFrame(listrating17)
            grating17 = lr17.iloc[6]
            self.ids.gainlblrt17.text = grating17.to_string()[1:]

            listrating18 = []
            for td in gainerhtml18:
                grate18 = td.find('span')
                listrating18.append(grate18)
            lr18 = pd.DataFrame(listrating18)
            grating18 = lr18.iloc[6]
            self.ids.gainlblrt18.text = grating18.to_string()[1:]

            listrating19 = []
            for td in gainerhtml19:
                grate19 = td.find('span')
                listrating19.append(grate19)
            lr19 = pd.DataFrame(listrating19)
            grating19 = lr19.iloc[6]
            self.ids.gainlblrt19.text = grating19.to_string()[1:]

            listrating20 = []
            for td in gainerhtml20:
                grate20 = td.find('span')
                listrating20.append(grate20)
            lr20 = pd.DataFrame(listrating20)
            grating20 = lr20.iloc[6]
            self.ids.gainlblrt20.text = grating20.to_string()[1:]

            listrating21 = []
            for td in gainerhtml21:
                grate21 = td.find('span')
                listrating21.append(grate21)
            lr21 = pd.DataFrame(listrating21)
            grating21 = lr21.iloc[6]
            self.ids.gainlblrt21.text = grating21.to_string()[1:]

            listrating22 = []
            for td in gainerhtml22:
                grate22 = td.find('span')
                listrating22.append(grate22)
            lr22 = pd.DataFrame(listrating22)
            grating22 = lr22.iloc[6]
            self.ids.gainlblrt22.text = grating22.to_string()[1:]

            listrating23 = []
            for td in gainerhtml23:
                grate23 = td.find('span')
                listrating23.append(grate23)
            lr23 = pd.DataFrame(listrating23)
            grating23 = lr23.iloc[6]
            self.ids.gainlblrt23.text = grating23.to_string()[1:]

            listrating24 = []
            for td in gainerhtml24:
                grate24 = td.find('span')
                listrating24.append(grate24)
            lr24 = pd.DataFrame(listrating24)
            grating24 = lr24.iloc[6]
            self.ids.gainlblrt24.text = grating24.to_string()[1:]

            listrating25 = []
            for td in gainerhtml25:
                grate25 = td.find('span')
                listrating25.append(grate25)
            lr25 = pd.DataFrame(listrating25)
            grating25 = lr25.iloc[6]
            self.ids.gainlblrt25.text = grating25.to_string()[1:]

            listrating26 = []
            for td in gainerhtml26:
                grate26 = td.find('span')
                listrating26.append(grate26)
            lr26 = pd.DataFrame(listrating26)
            grating26 = lr26.iloc[6]
            self.ids.gainlblrt26.text = grating26.to_string()[1:]

            listrating27 = []
            for td in gainerhtml27:
                grate27 = td.find('span')
                listrating27.append(grate27)
            lr27 = pd.DataFrame(listrating27)
            grating27 = lr27.iloc[6]
            self.ids.gainlblrt27.text = grating27.to_string()[1:]

            listrating28 = []
            for td in gainerhtml28:
                grate28 = td.find('span')
                listrating28.append(grate28)
            lr28 = pd.DataFrame(listrating28)
            grating28 = lr28.iloc[6]
            self.ids.gainlblrt28.text = grating28.to_string()[1:]

            listrating29 = []
            for td in gainerhtml29:
                grate29 = td.find('span')
                listrating29.append(grate29)
            lr29 = pd.DataFrame(listrating29)
            grating29 = lr29.iloc[6]
            self.ids.gainlblrt29.text = grating29.to_string()[1:]

            listrating30 = []
            for td in gainerhtml30:
                grate30 = td.find('span')
                listrating30.append(grate30)
            lr30 = pd.DataFrame(listrating30)
            grating30 = lr30.iloc[6]
            self.ids.gainlblrt30.text = grating30.to_string()[1:]

            listrating31 = []
            for td in gainerhtml31:
                grate31 = td.find('span')
                listrating31.append(grate31)
            lr31 = pd.DataFrame(listrating31)
            grating31 = lr31.iloc[6]
            self.ids.gainlblrt31.text = grating31.to_string()[1:]

            listrating32 = []
            for td in gainerhtml32:
                grate32 = td.find('span')
                listrating32.append(grate32)
            lr32 = pd.DataFrame(listrating32)
            grating32 = lr32.iloc[6]
            self.ids.gainlblrt32.text = grating32.to_string()[1:]

            listrating33 = []
            for td in gainerhtml33:
                grate33 = td.find('span')
                listrating33.append(grate33)
            lr33 = pd.DataFrame(listrating33)
            grating33 = lr33.iloc[6]
            self.ids.gainlblrt33.text = grating33.to_string()[1:]

            listrating34 = []
            for td in gainerhtml34:
                grate34 = td.find('span')
                listrating34.append(grate34)
            lr34 = pd.DataFrame(listrating34)
            grating34 = lr34.iloc[6]
            self.ids.gainlblrt34.text = grating34.to_string()[1:]

            listrating35 = []
            for td in gainerhtml35:
                grate35 = td.find('span')
                listrating35.append(grate35)
            lr35 = pd.DataFrame(listrating35)
            grating35 = lr35.iloc[6]
            self.ids.gainlblrt35.text = grating35.to_string()[1:]

            listrating36 = []
            for td in gainerhtml36:
                grate36 = td.find('span')
                listrating36.append(grate36)
            lr36 = pd.DataFrame(listrating36)
            grating36 = lr36.iloc[6]
            self.ids.gainlblrt36.text = grating36.to_string()[1:]

            listrating37 = []
            for td in gainerhtml37:
                grate37 = td.find('span')
                listrating37.append(grate37)
            lr37 = pd.DataFrame(listrating37)
            grating37 = lr37.iloc[6]
            self.ids.gainlblrt37.text = grating37.to_string()[1:]

            listrating38 = []
            for td in gainerhtml38:
                grate38 = td.find('span')
                listrating38.append(grate38)
            lr38 = pd.DataFrame(listrating38)
            grating38 = lr38.iloc[6]
            self.ids.gainlblrt38.text = grating38.to_string()[1:]

            listrating39 = []
            for td in gainerhtml39:
                grate39 = td.find('span')
                listrating39.append(grate39)
            lr39 = pd.DataFrame(listrating39)
            grating39 = lr39.iloc[6]
            self.ids.gainlblrt39.text = grating39.to_string()[1:]

            listrating40 = []
            for td in gainerhtml40:
                grate40 = td.find('span')
                listrating40.append(grate40)
            lr40 = pd.DataFrame(listrating40)
            grating40 = lr40.iloc[6]
            self.ids.gainlblrt40.text = grating40.to_string()[1:]

            listrating41 = []
            for td in gainerhtml41:
                grate41 = td.find('span')
                listrating41.append(grate41)
            lr41 = pd.DataFrame(listrating41)
            grating41 = lr41.iloc[6]
            self.ids.gainlblrt41.text = grating41.to_string()[1:]

            listrating42 = []
            for td in gainerhtml42:
                grate42 = td.find('span')
                listrating42.append(grate42)
            lr42 = pd.DataFrame(listrating42)
            grating42 = lr42.iloc[6]
            self.ids.gainlblrt42.text = grating42.to_string()[1:]

            listrating43 = []
            for td in gainerhtml43:
                grate43 = td.find('span')
                listrating43.append(grate43)
            lr43 = pd.DataFrame(listrating43)
            grating43 = lr43.iloc[6]
            self.ids.gainlblrt43.text = grating43.to_string()[1:]

            listrating44 = []
            for td in gainerhtml44:
                grate44 = td.find('span')
                listrating44.append(grate44)
            lr44 = pd.DataFrame(listrating44)
            grating44 = lr44.iloc[6]
            self.ids.gainlblrt44.text = grating44.to_string()[1:]

            listrating45 = []
            for td in gainerhtml45:
                grate45 = td.find('span')
                listrating45.append(grate45)
            lr45 = pd.DataFrame(listrating45)
            grating45 = lr45.iloc[6]
            self.ids.gainlblrt45.text = grating45.to_string()[1:]

            listrating46 = []
            for td in gainerhtml46:
                grate46 = td.find('span')
                listrating46.append(grate46)
            lr46 = pd.DataFrame(listrating46)
            grating46 = lr46.iloc[6]
            self.ids.gainlblrt46.text = grating46.to_string()[1:]

            listrating47 = []
            for td in gainerhtml47:
                grate47 = td.find('span')
                listrating47.append(grate47)
            lr47 = pd.DataFrame(listrating47)
            grating47 = lr47.iloc[6]
            self.ids.gainlblrt47.text = grating47.to_string()[1:]

            listrating48 = []
            for td in gainerhtml48:
                grate48 = td.find('span')
                listrating48.append(grate48)
            lr48 = pd.DataFrame(listrating48)
            grating48 = lr48.iloc[6]
            self.ids.gainlblrt48.text = grating48.to_string()[1:]

            listrating49 = []
            for td in gainerhtml49:
                grate49 = td.find('span')
                listrating49.append(grate49)
            lr49 = pd.DataFrame(listrating49)
            grating49 = lr49.iloc[6]
            self.ids.gainlblrt49.text = grating49.to_string()[1:]

            self.ids.q.text = '1.'
            self.ids.w.text = '2.'
            self.ids.e.text = '3.'
            self.ids.r.text = '4.'
            self.ids.t.text = '5.'
            self.ids.y.text = '6.'
            self.ids.u.text = '7.'
            self.ids.i.text = '8.'
            self.ids.o.text = '9.'
            self.ids.p.text = '10.'
            self.ids.a.text = '11.'
            self.ids.s.text = '12.'
            self.ids.d.text = '13.'
            self.ids.f.text = '14.'
            self.ids.g.text = '15.'
            self.ids.h.text = '16.'
            self.ids.j.text = '17.'
            self.ids.k.text = '18.'
            self.ids.l.text = '19.'
            self.ids.z.text = '20.'
            self.ids.x.text = '21.'
            self.ids.c.text = '22.'
            self.ids.v.text = '23.'
            self.ids.b.text = '24.'
            self.ids.n.text = '25.'
            self.ids.qq.text = '26.'
            self.ids.ww.text = '27.'
            self.ids.ee.text = '28.'
            self.ids.rr.text = '29.'
            self.ids.tt.text = '30.'
            self.ids.yy.text = '31.'
            self.ids.uu.text = '32.'
            self.ids.ii.text = '33.'
            self.ids.oo.text = '34.'
            self.ids.pp.text = '35.'
            self.ids.aa.text = '36.'
            self.ids.ss.text = '37.'
            self.ids.dd.text = '38.'
            self.ids.ff.text = '39.'
            self.ids.gg.text = '40.'
            self.ids.hh.text = '41.'
            self.ids.jj.text = '42.'
            self.ids.kk.text = '43.'
            self.ids.ll.text = '44.'
            self.ids.zz.text = '45.'
            self.ids.xx.text = '46.'
            self.ids.cc.text = '47.'
            self.ids.vv.text = '48.'
            self.ids.bb.text = '49.'
            self.ids.nn.text = '50.'

    def losref(self):
            urllosers = requests.get('https://www.tradingview.com/markets/stocks-usa/market-movers-losers/')
            souploserstick = bs4.BeautifulSoup(urllosers.text, features="html.parser")

            loserhtml0 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[0]
            loserhtml1 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[1]
            loserhtml2 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[2]
            loserhtml3 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[3]
            loserhtml4 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[4]
            loserhtml5 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[5]
            loserhtml6 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[6]
            loserhtml7 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[7]
            loserhtml8 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[8]
            loserhtml9 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[9]
            loserhtml10 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[10]
            loserhtml11 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[11]
            loserhtml12 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[12]
            loserhtml13 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[13]
            loserhtml14 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[14]
            loserhtml15 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[15]
            loserhtml16 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[16]
            loserhtml17 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[17]
            loserhtml18 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[18]
            loserhtml19 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[19]
            loserhtml20 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[20]
            loserhtml21 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[21]
            loserhtml22 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[22]
            loserhtml23 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[23]
            loserhtml24 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[24]
            loserhtml25 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[25]
            loserhtml26 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[26]
            loserhtml27 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[27]
            loserhtml28 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[28]
            loserhtml29 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[29]
            loserhtml30 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[30]
            loserhtml31 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[31]
            loserhtml32 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[32]
            loserhtml33 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[33]
            loserhtml34 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[34]
            loserhtml35 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[35]
            loserhtml36 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[36]
            loserhtml37 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[37]
            loserhtml38 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[38]
            loserhtml39 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[39]
            loserhtml40 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[40]
            loserhtml41 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[41]
            loserhtml42 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[42]
            loserhtml43 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[43]
            loserhtml44 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[44]
            loserhtml45 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[45]
            loserhtml46 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[46]
            loserhtml47 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[47]
            loserhtml48 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[48]
            loserhtml49 = souploserstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[49]

            loserstick0 = loserhtml0.find('a').text
            loserstick1 = loserhtml1.find('a').text
            loserstick2 = loserhtml2.find('a').text
            loserstick3 = loserhtml3.find('a').text
            loserstick4 = loserhtml4.find('a').text
            loserstick5 = loserhtml5.find('a').text
            loserstick6 = loserhtml6.find('a').text
            loserstick7 = loserhtml7.find('a').text
            loserstick8 = loserhtml8.find('a').text
            loserstick9 = loserhtml9.find('a').text
            loserstick10 = loserhtml10.find('a').text
            loserstick11 = loserhtml11.find('a').text
            loserstick12 = loserhtml12.find('a').text
            loserstick13 = loserhtml13.find('a').text
            loserstick14 = loserhtml14.find('a').text
            loserstick15 = loserhtml15.find('a').text
            loserstick16 = loserhtml16.find('a').text
            loserstick17 = loserhtml17.find('a').text
            loserstick18 = loserhtml18.find('a').text
            loserstick19 = loserhtml19.find('a').text
            loserstick20 = loserhtml20.find('a').text
            loserstick21 = loserhtml21.find('a').text
            loserstick22 = loserhtml22.find('a').text
            loserstick23 = loserhtml23.find('a').text
            loserstick24 = loserhtml24.find('a').text
            loserstick25 = loserhtml25.find('a').text
            loserstick26 = loserhtml26.find('a').text
            loserstick27 = loserhtml27.find('a').text
            loserstick28 = loserhtml28.find('a').text
            loserstick29 = loserhtml29.find('a').text
            loserstick30 = loserhtml30.find('a').text
            loserstick31 = loserhtml31.find('a').text
            loserstick32 = loserhtml32.find('a').text
            loserstick33 = loserhtml33.find('a').text
            loserstick34 = loserhtml34.find('a').text
            loserstick35 = loserhtml35.find('a').text
            loserstick36 = loserhtml36.find('a').text
            loserstick37 = loserhtml37.find('a').text
            loserstick38 = loserhtml38.find('a').text
            loserstick39 = loserhtml39.find('a').text
            loserstick40 = loserhtml40.find('a').text
            loserstick41 = loserhtml41.find('a').text
            loserstick42 = loserhtml42.find('a').text
            loserstick43 = loserhtml43.find('a').text
            loserstick44 = loserhtml44.find('a').text
            loserstick45 = loserhtml45.find('a').text
            loserstick46 = loserhtml46.find('a').text
            loserstick47 = loserhtml47.find('a').text
            loserstick48 = loserhtml48.find('a').text
            loserstick49 = loserhtml49.find('a').text
            
            self.ids.gainlbl0.text = loserstick0
            self.ids.gainlbl1.text = loserstick1
            self.ids.gainlbl2.text = loserstick2
            self.ids.gainlbl3.text = loserstick3
            self.ids.gainlbl4.text = loserstick4
            self.ids.gainlbl5.text = loserstick5
            self.ids.gainlbl6.text = loserstick6
            self.ids.gainlbl7.text = loserstick7
            self.ids.gainlbl8.text = loserstick8
            self.ids.gainlbl9.text = loserstick9
            self.ids.gainlbl10.text = loserstick10
            self.ids.gainlbl11.text = loserstick11
            self.ids.gainlbl12.text = loserstick12
            self.ids.gainlbl13.text = loserstick13
            self.ids.gainlbl14.text = loserstick14
            self.ids.gainlbl15.text = loserstick15
            self.ids.gainlbl16.text = loserstick16
            self.ids.gainlbl17.text = loserstick17
            self.ids.gainlbl18.text = loserstick18
            self.ids.gainlbl19.text = loserstick19
            self.ids.gainlbl20.text = loserstick20
            self.ids.gainlbl21.text = loserstick21
            self.ids.gainlbl22.text = loserstick22
            self.ids.gainlbl23.text = loserstick23
            self.ids.gainlbl24.text = loserstick24
            self.ids.gainlbl25.text = loserstick25
            self.ids.gainlbl26.text = loserstick26
            self.ids.gainlbl27.text = loserstick27
            self.ids.gainlbl28.text = loserstick28
            self.ids.gainlbl29.text = loserstick29
            self.ids.gainlbl30.text = loserstick30
            self.ids.gainlbl31.text = loserstick31
            self.ids.gainlbl32.text = loserstick32
            self.ids.gainlbl33.text = loserstick33
            self.ids.gainlbl34.text = loserstick34
            self.ids.gainlbl35.text = loserstick35
            self.ids.gainlbl36.text = loserstick36
            self.ids.gainlbl37.text = loserstick37
            self.ids.gainlbl38.text = loserstick38
            self.ids.gainlbl39.text = loserstick39
            self.ids.gainlbl40.text = loserstick40
            self.ids.gainlbl41.text = loserstick41
            self.ids.gainlbl42.text = loserstick42
            self.ids.gainlbl43.text = loserstick43
            self.ids.gainlbl44.text = loserstick44
            self.ids.gainlbl45.text = loserstick45
            self.ids.gainlbl46.text = loserstick46
            self.ids.gainlbl47.text = loserstick47
            self.ids.gainlbl48.text = loserstick48
            self.ids.gainlbl49.text = loserstick49

            self.ids.q.text = '1.'
            self.ids.w.text = '2.'
            self.ids.e.text = '3.'
            self.ids.r.text = '4.'
            self.ids.t.text = '5.'
            self.ids.y.text = '6.'
            self.ids.u.text = '7.'
            self.ids.i.text = '8.'
            self.ids.o.text = '9.'
            self.ids.p.text = '10.'
            self.ids.a.text = '11.'
            self.ids.s.text = '12.'
            self.ids.d.text = '13.'
            self.ids.f.text = '14.'
            self.ids.g.text = '15.'
            self.ids.h.text = '16.'
            self.ids.j.text = '17.'
            self.ids.k.text = '18.'
            self.ids.l.text = '19.'
            self.ids.z.text = '20.'
            self.ids.x.text = '21.'
            self.ids.c.text = '22.'
            self.ids.v.text = '23.'
            self.ids.b.text = '24.'
            self.ids.n.text = '25.'
            self.ids.qq.text = '26.'
            self.ids.ww.text = '27.'
            self.ids.ee.text = '28.'
            self.ids.rr.text = '29.'
            self.ids.tt.text = '30.'
            self.ids.yy.text = '31.'
            self.ids.uu.text = '32.'
            self.ids.ii.text = '33.'
            self.ids.oo.text = '34.'
            self.ids.pp.text = '35.'
            self.ids.aa.text = '36.'
            self.ids.ss.text = '37.'
            self.ids.dd.text = '38.'
            self.ids.ff.text = '39.'
            self.ids.gg.text = '40.'
            self.ids.hh.text = '41.'
            self.ids.jj.text = '42.'
            self.ids.kk.text = '43.'
            self.ids.ll.text = '44.'
            self.ids.zz.text = '45.'
            self.ids.xx.text = '46.'
            self.ids.cc.text = '47.'
            self.ids.vv.text = '48.'
            self.ids.bb.text = '49.'
            self.ids.nn.text = '50.'

            loslistrating0 = []
            for td in loserhtml0:
                lrate0 = td.find('span')
                loslistrating0.append(lrate0)
            loslr0 = pd.DataFrame(loslistrating0)
            lrating0 = loslr0.iloc[6]
            self.ids.gainlblrt0.text = lrating0.to_string()[1:]

            loslistrating1 = []
            for td in loserhtml1:
                lrate1 = td.find('span')
                loslistrating1.append(lrate1)
            loslr1 = pd.DataFrame(loslistrating1)
            lrating1 = loslr1.iloc[6]
            self.ids.gainlblrt1.text = lrating1.to_string()[1:]

            loslistrating2 = []
            for td in loserhtml2:
                lrate2 = td.find('span')
                loslistrating2.append(lrate2)
            loslr2 = pd.DataFrame(loslistrating2)
            lrating2 = loslr2.iloc[6]
            self.ids.gainlblrt2.text = lrating2.to_string()[1:]

            loslistrating3 = []
            for td in loserhtml3:
                lrate3 = td.find('span')
                loslistrating3.append(lrate3)
            loslr3 = pd.DataFrame(loslistrating3)
            lrating3 = loslr3.iloc[6]
            self.ids.gainlblrt3.text = lrating3.to_string()[1:]

            loslistrating4 = []
            for td in loserhtml4:
                lrate4 = td.find('span')
                loslistrating4.append(lrate4)
            loslr4 = pd.DataFrame(loslistrating4)
            lrating4 = loslr4.iloc[6]
            self.ids.gainlblrt4.text = lrating4.to_string()[1:]

            loslistrating5 = []
            for td in loserhtml5:
                lrate5 = td.find('span')
                loslistrating5.append(lrate5)
            loslr5 = pd.DataFrame(loslistrating5)
            lrating5 = loslr5.iloc[6]
            self.ids.gainlblrt5.text = lrating5.to_string()[1:]

            loslistrating6 = []
            for td in loserhtml6:
                lrate6 = td.find('span')
                loslistrating6.append(lrate6)
            loslr6 = pd.DataFrame(loslistrating6)
            lrating6 = loslr6.iloc[6]
            self.ids.gainlblrt6.text = lrating6.to_string()[1:]

            loslistrating7 = []
            for td in loserhtml7:
                lrate7 = td.find('span')
                loslistrating7.append(lrate7)
            loslr7 = pd.DataFrame(loslistrating7)
            lrating7 = loslr7.iloc[6]
            self.ids.gainlblrt7.text = lrating7.to_string()[1:]

            loslistrating8 = []
            for td in loserhtml8:
                lrate8 = td.find('span')
                loslistrating8.append(lrate8)
            loslr8 = pd.DataFrame(loslistrating8)
            lrating8 = loslr8.iloc[6]
            self.ids.gainlblrt8.text = lrating8.to_string()[1:]

            loslistrating9 = []
            for td in loserhtml9:
                lrate9 = td.find('span')
                loslistrating9.append(lrate9)
            loslr9 = pd.DataFrame(loslistrating9)
            lrating9 = loslr9.iloc[6]
            self.ids.gainlblrt9.text = lrating9.to_string()[1:]

            loslistrating10 = []
            for td in loserhtml10:
                lrate10 = td.find('span')
                loslistrating10.append(lrate10)
            loslr10 = pd.DataFrame(loslistrating10)
            lrating10 = loslr10.iloc[6]
            self.ids.gainlblrt10.text = lrating10.to_string()[1:]

            loslistrating11 = []
            for td in loserhtml11:
                lrate11 = td.find('span')
                loslistrating11.append(lrate11)
            loslr11 = pd.DataFrame(loslistrating11)
            lrating11 = loslr11.iloc[6]
            self.ids.gainlblrt11.text = lrating11.to_string()[1:]

            loslistrating12 = []
            for td in loserhtml12:
                lrate12 = td.find('span')
                loslistrating12.append(lrate12)
            loslr12 = pd.DataFrame(loslistrating12)
            lrating12 = loslr12.iloc[6]
            self.ids.gainlblrt12.text = lrating12.to_string()[1:]

            loslistrating13 = []
            for td in loserhtml13:
                lrate13 = td.find('span')
                loslistrating13.append(lrate13)
            loslr13 = pd.DataFrame(loslistrating13)
            lrating13 = loslr13.iloc[6]
            self.ids.gainlblrt13.text = lrating13.to_string()[1:]

            loslistrating14 = []
            for td in loserhtml14:
                lrate14 = td.find('span')
                loslistrating14.append(lrate14)
            loslr14 = pd.DataFrame(loslistrating14)
            lrating14 = loslr14.iloc[6]
            self.ids.gainlblrt14.text = lrating14.to_string()[1:]

            loslistrating15 = []
            for td in loserhtml15:
                lrate15 = td.find('span')
                loslistrating15.append(lrate15)
            loslr15 = pd.DataFrame(loslistrating15)
            lrating15 = loslr15.iloc[6]
            self.ids.gainlblrt15.text = lrating15.to_string()[1:]

            loslistrating16 = []
            for td in loserhtml16:
                lrate16 = td.find('span')
                loslistrating16.append(lrate16)
            loslr16 = pd.DataFrame(loslistrating16)
            lrating16 = loslr16.iloc[6]
            self.ids.gainlblrt16.text = lrating16.to_string()[1:]

            loslistrating17 = []
            for td in loserhtml17:
                lrate17 = td.find('span')
                loslistrating17.append(lrate17)
            loslr17 = pd.DataFrame(loslistrating17)
            lrating17 = loslr17.iloc[6]
            self.ids.gainlblrt17.text = lrating17.to_string()[1:]

            loslistrating18 = []
            for td in loserhtml18:
                lrate18 = td.find('span')
                loslistrating18.append(lrate18)
            loslr18 = pd.DataFrame(loslistrating18)
            lrating18 = loslr18.iloc[6]
            self.ids.gainlblrt18.text = lrating18.to_string()[1:]

            loslistrating19 = []
            for td in loserhtml19:
                lrate19 = td.find('span')
                loslistrating19.append(lrate19)
            loslr19 = pd.DataFrame(loslistrating19)
            lrating19 = loslr19.iloc[6]
            self.ids.gainlblrt19.text = lrating19.to_string()[1:]

            loslistrating20 = []
            for td in loserhtml20:
                lrate20 = td.find('span')
                loslistrating20.append(lrate20)
            loslr20 = pd.DataFrame(loslistrating20)
            lrating20 = loslr20.iloc[6]
            self.ids.gainlblrt20.text = lrating20.to_string()[1:]

            loslistrating21 = []
            for td in loserhtml21:
                lrate21 = td.find('span')
                loslistrating21.append(lrate21)
            loslr21 = pd.DataFrame(loslistrating21)
            lrating21 = loslr21.iloc[6]
            self.ids.gainlblrt21.text = lrating21.to_string()[1:]

            loslistrating22 = []
            for td in loserhtml22:
                lrate22 = td.find('span')
                loslistrating22.append(lrate22)
            loslr22 = pd.DataFrame(loslistrating22)
            lrating22 = loslr22.iloc[6]
            self.ids.gainlblrt22.text = lrating22.to_string()[1:]

            loslistrating23 = []
            for td in loserhtml23:
                lrate23 = td.find('span')
                loslistrating23.append(lrate23)
            loslr23 = pd.DataFrame(loslistrating23)
            lrating23 = loslr23.iloc[6]
            self.ids.gainlblrt23.text = lrating23.to_string()[1:]

            loslistrating24 = []
            for td in loserhtml24:
                lrate24 = td.find('span')
                loslistrating24.append(lrate24)
            loslr24 = pd.DataFrame(loslistrating24)
            lrating24 = loslr24.iloc[6]
            self.ids.gainlblrt24.text = lrating24.to_string()[1:]

            loslistrating25 = []
            for td in loserhtml25:
                lrate25 = td.find('span')
                loslistrating25.append(lrate25)
            loslr25 = pd.DataFrame(loslistrating25)
            lrating25 = loslr25.iloc[6]
            self.ids.gainlblrt25.text = lrating25.to_string()[1:]

            loslistrating26 = []
            for td in loserhtml26:
                lrate26 = td.find('span')
                loslistrating26.append(lrate26)
            loslr26 = pd.DataFrame(loslistrating26)
            lrating26 = loslr26.iloc[6]
            self.ids.gainlblrt26.text = lrating26.to_string()[1:]

            loslistrating27 = []
            for td in loserhtml27:
                lrate27 = td.find('span')
                loslistrating27.append(lrate27)
            loslr27 = pd.DataFrame(loslistrating27)
            lrating27 = loslr27.iloc[6]
            self.ids.gainlblrt27.text = lrating27.to_string()[1:]

            loslistrating28 = []
            for td in loserhtml28:
                lrate28 = td.find('span')
                loslistrating28.append(lrate28)
            loslr28 = pd.DataFrame(loslistrating28)
            lrating28 = loslr28.iloc[6]
            self.ids.gainlblrt28.text = lrating28.to_string()[1:]

            loslistrating29 = []
            for td in loserhtml29:
                lrate29 = td.find('span')
                loslistrating29.append(lrate29)
            loslr29 = pd.DataFrame(loslistrating29)
            lrating29 = loslr29.iloc[6]
            self.ids.gainlblrt29.text = lrating29.to_string()[1:]

            loslistrating30 = []
            for td in loserhtml30:
                lrate30 = td.find('span')
                loslistrating30.append(lrate30)
            loslr30 = pd.DataFrame(loslistrating30)
            lrating30 = loslr30.iloc[6]
            self.ids.gainlblrt30.text = lrating30.to_string()[1:]

            loslistrating31 = []
            for td in loserhtml31:
                lrate31 = td.find('span')
                loslistrating31.append(lrate31)
            loslr31 = pd.DataFrame(loslistrating31)
            lrating31 = loslr31.iloc[6]
            self.ids.gainlblrt31.text = lrating31.to_string()[1:]

            loslistrating32 = []
            for td in loserhtml32:
                lrate32 = td.find('span')
                loslistrating32.append(lrate32)
            loslr32 = pd.DataFrame(loslistrating32)
            lrating32 = loslr32.iloc[6]
            self.ids.gainlblrt32.text = lrating32.to_string()[1:]

            loslistrating33 = []
            for td in loserhtml33:
                lrate33 = td.find('span')
                loslistrating33.append(lrate33)
            loslr33 = pd.DataFrame(loslistrating33)
            lrating33 = loslr33.iloc[6]
            self.ids.gainlblrt33.text = lrating33.to_string()[1:]

            loslistrating34 = []
            for td in loserhtml34:
                lrate34 = td.find('span')
                loslistrating34.append(lrate34)
            loslr34 = pd.DataFrame(loslistrating34)
            lrating34 = loslr34.iloc[6]
            self.ids.gainlblrt34.text = lrating34.to_string()[1:]

            loslistrating35 = []
            for td in loserhtml35:
                lrate35 = td.find('span')
                loslistrating35.append(lrate35)
            loslr35 = pd.DataFrame(loslistrating35)
            lrating35 = loslr35.iloc[6]
            self.ids.gainlblrt35.text = lrating35.to_string()[1:]

            loslistrating36 = []
            for td in loserhtml36:
                lrate36 = td.find('span')
                loslistrating36.append(lrate36)
            loslr36 = pd.DataFrame(loslistrating36)
            lrating36 = loslr36.iloc[6]
            self.ids.gainlblrt36.text = lrating36.to_string()[1:]

            loslistrating37 = []
            for td in loserhtml37:
                lrate37 = td.find('span')
                loslistrating37.append(lrate37)
            loslr37 = pd.DataFrame(loslistrating37)
            lrating37 = loslr37.iloc[6]
            self.ids.gainlblrt37.text = lrating37.to_string()[1:]

            loslistrating38 = []
            for td in loserhtml38:
                lrate38 = td.find('span')
                loslistrating38.append(lrate38)
            loslr38 = pd.DataFrame(loslistrating38)
            lrating38 = loslr38.iloc[6]
            self.ids.gainlblrt38.text = lrating38.to_string()[1:]

            loslistrating39 = []
            for td in loserhtml39:
                lrate39 = td.find('span')
                loslistrating39.append(lrate39)
            loslr39 = pd.DataFrame(loslistrating39)
            lrating39 = loslr39.iloc[6]
            self.ids.gainlblrt39.text = lrating39.to_string()[1:]

            loslistrating40 = []
            for td in loserhtml40:
                lrate40 = td.find('span')
                loslistrating40.append(lrate40)
            loslr40 = pd.DataFrame(loslistrating40)
            lrating40 = loslr40.iloc[6]
            self.ids.gainlblrt40.text = lrating40.to_string()[1:]

            loslistrating41 = []
            for td in loserhtml41:
                lrate41 = td.find('span')
                loslistrating41.append(lrate41)
            loslr41 = pd.DataFrame(loslistrating41)
            lrating41 = loslr41.iloc[6]
            self.ids.gainlblrt41.text = lrating41.to_string()[1:]

            loslistrating42 = []
            for td in loserhtml42:
                lrate42 = td.find('span')
                loslistrating42.append(lrate42)
            loslr42 = pd.DataFrame(loslistrating42)
            lrating42 = loslr42.iloc[6]
            self.ids.gainlblrt42.text = lrating42.to_string()[1:]

            loslistrating43 = []
            for td in loserhtml43:
                lrate43 = td.find('span')
                loslistrating43.append(lrate43)
            loslr43 = pd.DataFrame(loslistrating43)
            lrating43 = loslr43.iloc[6]
            self.ids.gainlblrt43.text = lrating43.to_string()[1:]

            loslistrating44 = []
            for td in loserhtml44:
                lrate44 = td.find('span')
                loslistrating44.append(lrate44)
            loslr44 = pd.DataFrame(loslistrating44)
            lrating44 = loslr44.iloc[6]
            self.ids.gainlblrt44.text = lrating44.to_string()[1:]

            loslistrating45 = []
            for td in loserhtml45:
                lrate45 = td.find('span')
                loslistrating45.append(lrate45)
            loslr45 = pd.DataFrame(loslistrating45)
            lrating45 = loslr45.iloc[6]
            self.ids.gainlblrt45.text = lrating45.to_string()[1:]

            loslistrating46 = []
            for td in loserhtml46:
                lrate46 = td.find('span')
                loslistrating46.append(lrate46)
            loslr46 = pd.DataFrame(loslistrating46)
            lrating46 = loslr46.iloc[6]
            self.ids.gainlblrt46.text = lrating46.to_string()[1:]

            loslistrating47 = []
            for td in loserhtml47:
                lrate47 = td.find('span')
                loslistrating47.append(lrate47)
            loslr47 = pd.DataFrame(loslistrating47)
            lrating47 = loslr47.iloc[6]
            self.ids.gainlblrt47.text = lrating47.to_string()[1:]

            loslistrating48 = []
            for td in loserhtml48:
                lrate48 = td.find('span')
                loslistrating48.append(lrate48)
            loslr48 = pd.DataFrame(loslistrating48)
            lrating48 = loslr48.iloc[6]
            self.ids.gainlblrt48.text = lrating48.to_string()[1:]

            loslistrating49 = []
            for td in loserhtml49:
                lrate49 = td.find('span')
                loslistrating49.append(lrate49)
            loslr49 = pd.DataFrame(loslistrating49)
            lrating49 = loslr49.iloc[6]
            self.ids.gainlblrt49.text = lrating49.to_string()[1:]
            
    def activeref(self):

            urlgainers = requests.get('https://www.tradingview.com/markets/stocks-usa/market-movers-active/')
            soupgainerstick = bs4.BeautifulSoup(urlgainers.text, features="html.parser")

            gainerhtml0 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[0]
            gainerhtml1 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[1]
            gainerhtml2 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[2]
            gainerhtml3 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[3]
            gainerhtml4 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[4]
            gainerhtml5 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[5]
            gainerhtml6 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[6]
            gainerhtml7 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[7]
            gainerhtml8 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[8]
            gainerhtml9 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[9]
            gainerhtml10 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[10]
            gainerhtml11 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[11]
            gainerhtml12 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[12]
            gainerhtml13 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[13]
            gainerhtml14 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[14]
            gainerhtml15 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[15]
            gainerhtml16 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[16]
            gainerhtml17 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[17]
            gainerhtml18 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[18]
            gainerhtml19 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[19]
            gainerhtml20 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[20]
            gainerhtml21 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[21]
            gainerhtml22 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[22]
            gainerhtml23 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[23]
            gainerhtml24 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[24]
            gainerhtml25 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[25]
            gainerhtml26 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[26]
            gainerhtml27 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[27]
            gainerhtml28 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[28]
            gainerhtml29 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[29]
            gainerhtml30 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[30]
            gainerhtml31 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[31]
            gainerhtml32 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[32]
            gainerhtml33 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[33]
            gainerhtml34 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[34]
            gainerhtml35 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[35]
            gainerhtml36 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[36]
            gainerhtml37 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[37]
            gainerhtml38 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[38]
            gainerhtml39 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[39]
            gainerhtml40 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[40]
            gainerhtml41 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[41]
            gainerhtml42 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[42]
            gainerhtml43 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[43]
            gainerhtml44 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[44]
            gainerhtml45 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[45]
            gainerhtml46 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[46]
            gainerhtml47 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[47]
            gainerhtml48 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[48]
            gainerhtml49 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[49]

            gainerstick0 = gainerhtml0.find('a').text
            gainerstick1 = gainerhtml1.find('a').text
            gainerstick2 = gainerhtml2.find('a').text
            gainerstick3 = gainerhtml3.find('a').text
            gainerstick4 = gainerhtml4.find('a').text
            gainerstick5 = gainerhtml5.find('a').text
            gainerstick6 = gainerhtml6.find('a').text
            gainerstick7 = gainerhtml7.find('a').text
            gainerstick8 = gainerhtml8.find('a').text
            gainerstick9 = gainerhtml9.find('a').text
            gainerstick10 = gainerhtml10.find('a').text
            gainerstick11 = gainerhtml11.find('a').text
            gainerstick12 = gainerhtml12.find('a').text
            gainerstick13 = gainerhtml13.find('a').text
            gainerstick14 = gainerhtml14.find('a').text
            gainerstick15 = gainerhtml15.find('a').text
            gainerstick16 = gainerhtml16.find('a').text
            gainerstick17 = gainerhtml17.find('a').text
            gainerstick18 = gainerhtml18.find('a').text
            gainerstick19 = gainerhtml19.find('a').text
            gainerstick20 = gainerhtml20.find('a').text
            gainerstick21 = gainerhtml21.find('a').text
            gainerstick22 = gainerhtml22.find('a').text
            gainerstick23 = gainerhtml23.find('a').text
            gainerstick24 = gainerhtml24.find('a').text
            gainerstick25 = gainerhtml25.find('a').text
            gainerstick26 = gainerhtml26.find('a').text
            gainerstick27 = gainerhtml27.find('a').text
            gainerstick28 = gainerhtml28.find('a').text
            gainerstick29 = gainerhtml29.find('a').text
            gainerstick30 = gainerhtml30.find('a').text
            gainerstick31 = gainerhtml31.find('a').text
            gainerstick32 = gainerhtml32.find('a').text
            gainerstick33 = gainerhtml33.find('a').text
            gainerstick34 = gainerhtml34.find('a').text
            gainerstick35 = gainerhtml35.find('a').text
            gainerstick36 = gainerhtml36.find('a').text
            gainerstick37 = gainerhtml37.find('a').text
            gainerstick38 = gainerhtml38.find('a').text
            gainerstick39 = gainerhtml39.find('a').text
            gainerstick40 = gainerhtml40.find('a').text
            gainerstick41 = gainerhtml41.find('a').text
            gainerstick42 = gainerhtml42.find('a').text
            gainerstick43 = gainerhtml43.find('a').text
            gainerstick44 = gainerhtml44.find('a').text
            gainerstick45 = gainerhtml45.find('a').text
            gainerstick46 = gainerhtml46.find('a').text
            gainerstick47 = gainerhtml47.find('a').text
            gainerstick48 = gainerhtml48.find('a').text
            gainerstick49 = gainerhtml49.find('a').text
            
            self.ids.gainlbl0.text = gainerstick0
            self.ids.gainlbl1.text = gainerstick1
            self.ids.gainlbl2.text = gainerstick2
            self.ids.gainlbl3.text = gainerstick3
            self.ids.gainlbl4.text = gainerstick4
            self.ids.gainlbl5.text = gainerstick5
            self.ids.gainlbl6.text = gainerstick6
            self.ids.gainlbl7.text = gainerstick7
            self.ids.gainlbl8.text = gainerstick8
            self.ids.gainlbl9.text = gainerstick9
            self.ids.gainlbl10.text = gainerstick10
            self.ids.gainlbl11.text = gainerstick11
            self.ids.gainlbl12.text = gainerstick12
            self.ids.gainlbl13.text = gainerstick13
            self.ids.gainlbl14.text = gainerstick14
            self.ids.gainlbl15.text = gainerstick15
            self.ids.gainlbl16.text = gainerstick16
            self.ids.gainlbl17.text = gainerstick17
            self.ids.gainlbl18.text = gainerstick18
            self.ids.gainlbl19.text = gainerstick19
            self.ids.gainlbl20.text = gainerstick20
            self.ids.gainlbl21.text = gainerstick21
            self.ids.gainlbl22.text = gainerstick22
            self.ids.gainlbl23.text = gainerstick23
            self.ids.gainlbl24.text = gainerstick24
            self.ids.gainlbl25.text = gainerstick25
            self.ids.gainlbl26.text = gainerstick26
            self.ids.gainlbl27.text = gainerstick27
            self.ids.gainlbl28.text = gainerstick28
            self.ids.gainlbl29.text = gainerstick29
            self.ids.gainlbl30.text = gainerstick30
            self.ids.gainlbl31.text = gainerstick31
            self.ids.gainlbl32.text = gainerstick32
            self.ids.gainlbl33.text = gainerstick33
            self.ids.gainlbl34.text = gainerstick34
            self.ids.gainlbl35.text = gainerstick35
            self.ids.gainlbl36.text = gainerstick36
            self.ids.gainlbl37.text = gainerstick37
            self.ids.gainlbl38.text = gainerstick38
            self.ids.gainlbl39.text = gainerstick39
            self.ids.gainlbl40.text = gainerstick40
            self.ids.gainlbl41.text = gainerstick41
            self.ids.gainlbl42.text = gainerstick42
            self.ids.gainlbl43.text = gainerstick43
            self.ids.gainlbl44.text = gainerstick44
            self.ids.gainlbl45.text = gainerstick45
            self.ids.gainlbl46.text = gainerstick46
            self.ids.gainlbl47.text = gainerstick47
            self.ids.gainlbl48.text = gainerstick48
            self.ids.gainlbl49.text = gainerstick49

            self.ids.q.text = '1.'
            self.ids.w.text = '2.'
            self.ids.e.text = '3.'
            self.ids.r.text = '4.'
            self.ids.t.text = '5.'
            self.ids.y.text = '6.'
            self.ids.u.text = '7.'
            self.ids.i.text = '8.'
            self.ids.o.text = '9.'
            self.ids.p.text = '10.'
            self.ids.a.text = '11.'
            self.ids.s.text = '12.'
            self.ids.d.text = '13.'
            self.ids.f.text = '14.'
            self.ids.g.text = '15.'
            self.ids.h.text = '16.'
            self.ids.j.text = '17.'
            self.ids.k.text = '18.'
            self.ids.l.text = '19.'
            self.ids.z.text = '20.'
            self.ids.x.text = '21.'
            self.ids.c.text = '22.'
            self.ids.v.text = '23.'
            self.ids.b.text = '24.'
            self.ids.n.text = '25.'
            self.ids.qq.text = '26.'
            self.ids.ww.text = '27.'
            self.ids.ee.text = '28.'
            self.ids.rr.text = '29.'
            self.ids.tt.text = '30.'
            self.ids.yy.text = '31.'
            self.ids.uu.text = '32.'
            self.ids.ii.text = '33.'
            self.ids.oo.text = '34.'
            self.ids.pp.text = '35.'
            self.ids.aa.text = '36.'
            self.ids.ss.text = '37.'
            self.ids.dd.text = '38.'
            self.ids.ff.text = '39.'
            self.ids.gg.text = '40.'
            self.ids.hh.text = '41.'
            self.ids.jj.text = '42.'
            self.ids.kk.text = '43.'
            self.ids.ll.text = '44.'
            self.ids.zz.text = '45.'
            self.ids.xx.text = '46.'
            self.ids.cc.text = '47.'
            self.ids.vv.text = '48.'
            self.ids.bb.text = '49.'
            self.ids.nn.text = '50.'

            listrating0 = []
            for td in gainerhtml0:
                grate0 = td.find('span')
                listrating0.append(grate0)
            lr0 = pd.DataFrame(listrating0)
            grating0 = lr0.iloc[6]
            self.ids.gainlblrt0.text = grating0.to_string()[1:]

            listrating1 = []
            for td in gainerhtml1:
                grate1 = td.find('span')
                listrating1.append(grate1)
            lr1 = pd.DataFrame(listrating1)
            grating1 = lr1.iloc[6]
            self.ids.gainlblrt1.text = grating1.to_string()[1:]

            listrating2 = []
            for td in gainerhtml2:
                grate2 = td.find('span')
                listrating2.append(grate2)
            lr2 = pd.DataFrame(listrating2)
            grating2 = lr2.iloc[6]
            self.ids.gainlblrt2.text = grating2.to_string()[1:]

            listrating3 = []
            for td in gainerhtml3:
                grate3 = td.find('span')
                listrating3.append(grate3)
            lr3 = pd.DataFrame(listrating3)
            grating3 = lr3.iloc[6]
            self.ids.gainlblrt3.text = grating3.to_string()[1:]

            listrating4 = []
            for td in gainerhtml4:
                grate4 = td.find('span')
                listrating4.append(grate4)
            lr4 = pd.DataFrame(listrating4)
            grating4 = lr4.iloc[6]
            self.ids.gainlblrt4.text = grating4.to_string()[1:]

            listrating5 = []
            for td in gainerhtml5:
                grate5 = td.find('span')
                listrating5.append(grate5)
            lr5 = pd.DataFrame(listrating5)
            grating5 = lr5.iloc[6]
            self.ids.gainlblrt5.text = grating5.to_string()[1:]

            listrating6 = []
            for td in gainerhtml6:
                grate6 = td.find('span')
                listrating6.append(grate6)
            lr6 = pd.DataFrame(listrating6)
            grating6 = lr6.iloc[6]
            self.ids.gainlblrt6.text = grating6.to_string()[1:]

            listrating7 = []
            for td in gainerhtml7:
                grate7 = td.find('span')
                listrating7.append(grate7)
            lr7 = pd.DataFrame(listrating7)
            grating7 = lr7.iloc[6]
            self.ids.gainlblrt7.text = grating7.to_string()[1:]

            listrating8 = []
            for td in gainerhtml8:
                grate8 = td.find('span')
                listrating8.append(grate8)
            lr8 = pd.DataFrame(listrating8)
            grating8 = lr8.iloc[6]
            self.ids.gainlblrt8.text = grating8.to_string()[1:]

            listrating9 = []
            for td in gainerhtml9:
                grate9 = td.find('span')
                listrating9.append(grate9)
            lr9 = pd.DataFrame(listrating9)
            grating9 = lr9.iloc[6]
            self.ids.gainlblrt9.text = grating9.to_string()[1:]

            listrating10 = []
            for td in gainerhtml10:
                grate10 = td.find('span')
                listrating10.append(grate10)
            lr10 = pd.DataFrame(listrating10)
            grating10 = lr10.iloc[6]
            self.ids.gainlblrt10.text = grating10.to_string()[1:]

            listrating11 = []
            for td in gainerhtml11:
                grate11 = td.find('span')
                listrating11.append(grate11)
            lr11 = pd.DataFrame(listrating11)
            grating11 = lr11.iloc[6]
            self.ids.gainlblrt11.text = grating11.to_string()[1:]

            listrating12 = []
            for td in gainerhtml12:
                grate12 = td.find('span')
                listrating12.append(grate12)
            lr12 = pd.DataFrame(listrating12)
            grating12 = lr12.iloc[6]
            self.ids.gainlblrt12.text = grating12.to_string()[1:]

            listrating13 = []
            for td in gainerhtml13:
                grate13 = td.find('span')
                listrating13.append(grate13)
            lr13 = pd.DataFrame(listrating13)
            grating13 = lr13.iloc[6]
            self.ids.gainlblrt13.text = grating13.to_string()[1:]

            listrating14 = []
            for td in gainerhtml14:
                grate14 = td.find('span')
                listrating14.append(grate14)
            lr14 = pd.DataFrame(listrating14)
            grating14 = lr14.iloc[6]
            self.ids.gainlblrt14.text = grating14.to_string()[1:]

            listrating15 = []
            for td in gainerhtml15:
                grate15 = td.find('span')
                listrating15.append(grate15)
            lr15 = pd.DataFrame(listrating15)
            grating15 = lr15.iloc[6]
            self.ids.gainlblrt15.text = grating15.to_string()[1:]

            listrating16 = []
            for td in gainerhtml16:
                grate16 = td.find('span')
                listrating16.append(grate16)
            lr16 = pd.DataFrame(listrating16)
            grating16 = lr16.iloc[6]
            self.ids.gainlblrt16.text = grating16.to_string()[1:]

            listrating17 = []
            for td in gainerhtml17:
                grate17 = td.find('span')
                listrating17.append(grate17)
            lr17 = pd.DataFrame(listrating17)
            grating17 = lr17.iloc[6]
            self.ids.gainlblrt17.text = grating17.to_string()[1:]

            listrating18 = []
            for td in gainerhtml18:
                grate18 = td.find('span')
                listrating18.append(grate18)
            lr18 = pd.DataFrame(listrating18)
            grating18 = lr18.iloc[6]
            self.ids.gainlblrt18.text = grating18.to_string()[1:]

            listrating19 = []
            for td in gainerhtml19:
                grate19 = td.find('span')
                listrating19.append(grate19)
            lr19 = pd.DataFrame(listrating19)
            grating19 = lr19.iloc[6]
            self.ids.gainlblrt19.text = grating19.to_string()[1:]

            listrating20 = []
            for td in gainerhtml20:
                grate20 = td.find('span')
                listrating20.append(grate20)
            lr20 = pd.DataFrame(listrating20)
            grating20 = lr20.iloc[6]
            self.ids.gainlblrt20.text = grating20.to_string()[1:]

            listrating21 = []
            for td in gainerhtml21:
                grate21 = td.find('span')
                listrating21.append(grate21)
            lr21 = pd.DataFrame(listrating21)
            grating21 = lr21.iloc[6]
            self.ids.gainlblrt21.text = grating21.to_string()[1:]

            listrating22 = []
            for td in gainerhtml22:
                grate22 = td.find('span')
                listrating22.append(grate22)
            lr22 = pd.DataFrame(listrating22)
            grating22 = lr22.iloc[6]
            self.ids.gainlblrt22.text = grating22.to_string()[1:]

            listrating23 = []
            for td in gainerhtml23:
                grate23 = td.find('span')
                listrating23.append(grate23)
            lr23 = pd.DataFrame(listrating23)
            grating23 = lr23.iloc[6]
            self.ids.gainlblrt23.text = grating23.to_string()[1:]

            listrating24 = []
            for td in gainerhtml24:
                grate24 = td.find('span')
                listrating24.append(grate24)
            lr24 = pd.DataFrame(listrating24)
            grating24 = lr24.iloc[6]
            self.ids.gainlblrt24.text = grating24.to_string()[1:]

            listrating25 = []
            for td in gainerhtml25:
                grate25 = td.find('span')
                listrating25.append(grate25)
            lr25 = pd.DataFrame(listrating25)
            grating25 = lr25.iloc[6]
            self.ids.gainlblrt25.text = grating25.to_string()[1:]

            listrating26 = []
            for td in gainerhtml26:
                grate26 = td.find('span')
                listrating26.append(grate26)
            lr26 = pd.DataFrame(listrating26)
            grating26 = lr26.iloc[6]
            self.ids.gainlblrt26.text = grating26.to_string()[1:]

            listrating27 = []
            for td in gainerhtml27:
                grate27 = td.find('span')
                listrating27.append(grate27)
            lr27 = pd.DataFrame(listrating27)
            grating27 = lr27.iloc[6]
            self.ids.gainlblrt27.text = grating27.to_string()[1:]

            listrating28 = []
            for td in gainerhtml28:
                grate28 = td.find('span')
                listrating28.append(grate28)
            lr28 = pd.DataFrame(listrating28)
            grating28 = lr28.iloc[6]
            self.ids.gainlblrt28.text = grating28.to_string()[1:]

            listrating29 = []
            for td in gainerhtml29:
                grate29 = td.find('span')
                listrating29.append(grate29)
            lr29 = pd.DataFrame(listrating29)
            grating29 = lr29.iloc[6]
            self.ids.gainlblrt29.text = grating29.to_string()[1:]

            listrating30 = []
            for td in gainerhtml30:
                grate30 = td.find('span')
                listrating30.append(grate30)
            lr30 = pd.DataFrame(listrating30)
            grating30 = lr30.iloc[6]
            self.ids.gainlblrt30.text = grating30.to_string()[1:]

            listrating31 = []
            for td in gainerhtml31:
                grate31 = td.find('span')
                listrating31.append(grate31)
            lr31 = pd.DataFrame(listrating31)
            grating31 = lr31.iloc[6]
            self.ids.gainlblrt31.text = grating31.to_string()[1:]

            listrating32 = []
            for td in gainerhtml32:
                grate32 = td.find('span')
                listrating32.append(grate32)
            lr32 = pd.DataFrame(listrating32)
            grating32 = lr32.iloc[6]
            self.ids.gainlblrt32.text = grating32.to_string()[1:]

            listrating33 = []
            for td in gainerhtml33:
                grate33 = td.find('span')
                listrating33.append(grate33)
            lr33 = pd.DataFrame(listrating33)
            grating33 = lr33.iloc[6]
            self.ids.gainlblrt33.text = grating33.to_string()[1:]

            listrating34 = []
            for td in gainerhtml34:
                grate34 = td.find('span')
                listrating34.append(grate34)
            lr34 = pd.DataFrame(listrating34)
            grating34 = lr34.iloc[6]
            self.ids.gainlblrt34.text = grating34.to_string()[1:]

            listrating35 = []
            for td in gainerhtml35:
                grate35 = td.find('span')
                listrating35.append(grate35)
            lr35 = pd.DataFrame(listrating35)
            grating35 = lr35.iloc[6]
            self.ids.gainlblrt35.text = grating35.to_string()[1:]

            listrating36 = []
            for td in gainerhtml36:
                grate36 = td.find('span')
                listrating36.append(grate36)
            lr36 = pd.DataFrame(listrating36)
            grating36 = lr36.iloc[6]
            self.ids.gainlblrt36.text = grating36.to_string()[1:]

            listrating37 = []
            for td in gainerhtml37:
                grate37 = td.find('span')
                listrating37.append(grate37)
            lr37 = pd.DataFrame(listrating37)
            grating37 = lr37.iloc[6]
            self.ids.gainlblrt37.text = grating37.to_string()[1:]

            listrating38 = []
            for td in gainerhtml38:
                grate38 = td.find('span')
                listrating38.append(grate38)
            lr38 = pd.DataFrame(listrating38)
            grating38 = lr38.iloc[6]
            self.ids.gainlblrt38.text = grating38.to_string()[1:]

            listrating39 = []
            for td in gainerhtml39:
                grate39 = td.find('span')
                listrating39.append(grate39)
            lr39 = pd.DataFrame(listrating39)
            grating39 = lr39.iloc[6]
            self.ids.gainlblrt39.text = grating39.to_string()[1:]

            listrating40 = []
            for td in gainerhtml40:
                grate40 = td.find('span')
                listrating40.append(grate40)
            lr40 = pd.DataFrame(listrating40)
            grating40 = lr40.iloc[6]
            self.ids.gainlblrt40.text = grating40.to_string()[1:]

            listrating41 = []
            for td in gainerhtml41:
                grate41 = td.find('span')
                listrating41.append(grate41)
            lr41 = pd.DataFrame(listrating41)
            grating41 = lr41.iloc[6]
            self.ids.gainlblrt41.text = grating41.to_string()[1:]

            listrating42 = []
            for td in gainerhtml42:
                grate42 = td.find('span')
                listrating42.append(grate42)
            lr42 = pd.DataFrame(listrating42)
            grating42 = lr42.iloc[6]
            self.ids.gainlblrt42.text = grating42.to_string()[1:]

            listrating43 = []
            for td in gainerhtml43:
                grate43 = td.find('span')
                listrating43.append(grate43)
            lr43 = pd.DataFrame(listrating43)
            grating43 = lr43.iloc[6]
            self.ids.gainlblrt43.text = grating43.to_string()[1:]

            listrating44 = []
            for td in gainerhtml44:
                grate44 = td.find('span')
                listrating44.append(grate44)
            lr44 = pd.DataFrame(listrating44)
            grating44 = lr44.iloc[6]
            self.ids.gainlblrt44.text = grating44.to_string()[1:]

            listrating45 = []
            for td in gainerhtml45:
                grate45 = td.find('span')
                listrating45.append(grate45)
            lr45 = pd.DataFrame(listrating45)
            grating45 = lr45.iloc[6]
            self.ids.gainlblrt45.text = grating45.to_string()[1:]

            listrating46 = []
            for td in gainerhtml46:
                grate46 = td.find('span')
                listrating46.append(grate46)
            lr46 = pd.DataFrame(listrating46)
            grating46 = lr46.iloc[6]
            self.ids.gainlblrt46.text = grating46.to_string()[1:]

            listrating47 = []
            for td in gainerhtml47:
                grate47 = td.find('span')
                listrating47.append(grate47)
            lr47 = pd.DataFrame(listrating47)
            grating47 = lr47.iloc[6]
            self.ids.gainlblrt47.text = grating47.to_string()[1:]

            listrating48 = []
            for td in gainerhtml48:
                grate48 = td.find('span')
                listrating48.append(grate48)
            lr48 = pd.DataFrame(listrating48)
            grating48 = lr48.iloc[6]
            self.ids.gainlblrt48.text = grating48.to_string()[1:]

            listrating49 = []
            for td in gainerhtml49:
                grate49 = td.find('span')
                listrating49.append(grate49)
            lr49 = pd.DataFrame(listrating49)
            grating49 = lr49.iloc[6]
            self.ids.gainlblrt49.text = grating49.to_string()[1:]

    def volaref(self):

            urlgainers = requests.get('https://www.tradingview.com/markets/stocks-usa/market-movers-most-volatile/')
            soupgainerstick = bs4.BeautifulSoup(urlgainers.text, features="html.parser")


            gainerhtml0 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[0]
            gainerhtml1 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[1]
            gainerhtml2 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[2]
            gainerhtml3 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[3]
            gainerhtml4 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[4]
            gainerhtml5 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[5]
            gainerhtml6 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[6]
            gainerhtml7 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[7]
            gainerhtml8 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[8]
            gainerhtml9 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[9]
            gainerhtml10 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[10]
            gainerhtml11 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[11]
            gainerhtml12 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[12]
            gainerhtml13 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[13]
            gainerhtml14 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[14]
            gainerhtml15 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[15]
            gainerhtml16 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[16]
            gainerhtml17 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[17]
            gainerhtml18 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[18]
            gainerhtml19 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[19]
            gainerhtml20 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[20]
            gainerhtml21 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[21]
            gainerhtml22 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[22]
            gainerhtml23 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[23]
            gainerhtml24 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[24]
            gainerhtml25 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[25]
            gainerhtml26 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[26]
            gainerhtml27 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[27]
            gainerhtml28 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[28]
            gainerhtml29 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[29]
            gainerhtml30 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[30]
            gainerhtml31 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[31]
            gainerhtml32 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[32]
            gainerhtml33 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[33]
            gainerhtml34 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[34]
            gainerhtml35 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[35]
            gainerhtml36 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[36]
            gainerhtml37 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[37]
            gainerhtml38 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[38]
            gainerhtml39 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[39]
            gainerhtml40 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[40]
            gainerhtml41 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[41]
            gainerhtml42 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[42]
            gainerhtml43 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[43]
            gainerhtml44 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[44]
            gainerhtml45 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[45]
            gainerhtml46 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[46]
            gainerhtml47 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[47]
            gainerhtml48 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[48]
            gainerhtml49 = soupgainerstick.find_all("tr",{'class': "tv-data-table__row tv-data-table__stroke tv-screener-table__result-row"})[49]

            gainerstick0 = gainerhtml0.find('a').text
            gainerstick1 = gainerhtml1.find('a').text
            gainerstick2 = gainerhtml2.find('a').text
            gainerstick3 = gainerhtml3.find('a').text
            gainerstick4 = gainerhtml4.find('a').text
            gainerstick5 = gainerhtml5.find('a').text
            gainerstick6 = gainerhtml6.find('a').text
            gainerstick7 = gainerhtml7.find('a').text
            gainerstick8 = gainerhtml8.find('a').text
            gainerstick9 = gainerhtml9.find('a').text
            gainerstick10 = gainerhtml10.find('a').text
            gainerstick11 = gainerhtml11.find('a').text
            gainerstick12 = gainerhtml12.find('a').text
            gainerstick13 = gainerhtml13.find('a').text
            gainerstick14 = gainerhtml14.find('a').text
            gainerstick15 = gainerhtml15.find('a').text
            gainerstick16 = gainerhtml16.find('a').text
            gainerstick17 = gainerhtml17.find('a').text
            gainerstick18 = gainerhtml18.find('a').text
            gainerstick19 = gainerhtml19.find('a').text
            gainerstick20 = gainerhtml20.find('a').text
            gainerstick21 = gainerhtml21.find('a').text
            gainerstick22 = gainerhtml22.find('a').text
            gainerstick23 = gainerhtml23.find('a').text
            gainerstick24 = gainerhtml24.find('a').text
            gainerstick25 = gainerhtml25.find('a').text
            gainerstick26 = gainerhtml26.find('a').text
            gainerstick27 = gainerhtml27.find('a').text
            gainerstick28 = gainerhtml28.find('a').text
            gainerstick29 = gainerhtml29.find('a').text
            gainerstick30 = gainerhtml30.find('a').text
            gainerstick31 = gainerhtml31.find('a').text
            gainerstick32 = gainerhtml32.find('a').text
            gainerstick33 = gainerhtml33.find('a').text
            gainerstick34 = gainerhtml34.find('a').text
            gainerstick35 = gainerhtml35.find('a').text
            gainerstick36 = gainerhtml36.find('a').text
            gainerstick37 = gainerhtml37.find('a').text
            gainerstick38 = gainerhtml38.find('a').text
            gainerstick39 = gainerhtml39.find('a').text
            gainerstick40 = gainerhtml40.find('a').text
            gainerstick41 = gainerhtml41.find('a').text
            gainerstick42 = gainerhtml42.find('a').text
            gainerstick43 = gainerhtml43.find('a').text
            gainerstick44 = gainerhtml44.find('a').text
            gainerstick45 = gainerhtml45.find('a').text
            gainerstick46 = gainerhtml46.find('a').text
            gainerstick47 = gainerhtml47.find('a').text
            gainerstick48 = gainerhtml48.find('a').text
            gainerstick49 = gainerhtml49.find('a').text
            
            self.ids.gainlbl0.text = gainerstick0
            self.ids.gainlbl1.text = gainerstick1
            self.ids.gainlbl2.text = gainerstick2
            self.ids.gainlbl3.text = gainerstick3
            self.ids.gainlbl4.text = gainerstick4
            self.ids.gainlbl5.text = gainerstick5
            self.ids.gainlbl6.text = gainerstick6
            self.ids.gainlbl7.text = gainerstick7
            self.ids.gainlbl8.text = gainerstick8
            self.ids.gainlbl9.text = gainerstick9
            self.ids.gainlbl10.text = gainerstick10
            self.ids.gainlbl11.text = gainerstick11
            self.ids.gainlbl12.text = gainerstick12
            self.ids.gainlbl13.text = gainerstick13
            self.ids.gainlbl14.text = gainerstick14
            self.ids.gainlbl15.text = gainerstick15
            self.ids.gainlbl16.text = gainerstick16
            self.ids.gainlbl17.text = gainerstick17
            self.ids.gainlbl18.text = gainerstick18
            self.ids.gainlbl19.text = gainerstick19
            self.ids.gainlbl20.text = gainerstick20
            self.ids.gainlbl21.text = gainerstick21
            self.ids.gainlbl22.text = gainerstick22
            self.ids.gainlbl23.text = gainerstick23
            self.ids.gainlbl24.text = gainerstick24
            self.ids.gainlbl25.text = gainerstick25
            self.ids.gainlbl26.text = gainerstick26
            self.ids.gainlbl27.text = gainerstick27
            self.ids.gainlbl28.text = gainerstick28
            self.ids.gainlbl29.text = gainerstick29
            self.ids.gainlbl30.text = gainerstick30
            self.ids.gainlbl31.text = gainerstick31
            self.ids.gainlbl32.text = gainerstick32
            self.ids.gainlbl33.text = gainerstick33
            self.ids.gainlbl34.text = gainerstick34
            self.ids.gainlbl35.text = gainerstick35
            self.ids.gainlbl36.text = gainerstick36
            self.ids.gainlbl37.text = gainerstick37
            self.ids.gainlbl38.text = gainerstick38
            self.ids.gainlbl39.text = gainerstick39
            self.ids.gainlbl40.text = gainerstick40
            self.ids.gainlbl41.text = gainerstick41
            self.ids.gainlbl42.text = gainerstick42
            self.ids.gainlbl43.text = gainerstick43
            self.ids.gainlbl44.text = gainerstick44
            self.ids.gainlbl45.text = gainerstick45
            self.ids.gainlbl46.text = gainerstick46
            self.ids.gainlbl47.text = gainerstick47
            self.ids.gainlbl48.text = gainerstick48
            self.ids.gainlbl49.text = gainerstick49

            self.ids.q.text = '1.'
            self.ids.w.text = '2.'
            self.ids.e.text = '3.'
            self.ids.r.text = '4.'
            self.ids.t.text = '5.'
            self.ids.y.text = '6.'
            self.ids.u.text = '7.'
            self.ids.i.text = '8.'
            self.ids.o.text = '9.'
            self.ids.p.text = '10.'
            self.ids.a.text = '11.'
            self.ids.s.text = '12.'
            self.ids.d.text = '13.'
            self.ids.f.text = '14.'
            self.ids.g.text = '15.'
            self.ids.h.text = '16.'
            self.ids.j.text = '17.'
            self.ids.k.text = '18.'
            self.ids.l.text = '19.'
            self.ids.z.text = '20.'
            self.ids.x.text = '21.'
            self.ids.c.text = '22.'
            self.ids.v.text = '23.'
            self.ids.b.text = '24.'
            self.ids.n.text = '25.'
            self.ids.qq.text = '26.'
            self.ids.ww.text = '27.'
            self.ids.ee.text = '28.'
            self.ids.rr.text = '29.'
            self.ids.tt.text = '30.'
            self.ids.yy.text = '31.'
            self.ids.uu.text = '32.'
            self.ids.ii.text = '33.'
            self.ids.oo.text = '34.'
            self.ids.pp.text = '35.'
            self.ids.aa.text = '36.'
            self.ids.ss.text = '37.'
            self.ids.dd.text = '38.'
            self.ids.ff.text = '39.'
            self.ids.gg.text = '40.'
            self.ids.hh.text = '41.'
            self.ids.jj.text = '42.'
            self.ids.kk.text = '43.'
            self.ids.ll.text = '44.'
            self.ids.zz.text = '45.'
            self.ids.xx.text = '46.'
            self.ids.cc.text = '47.'
            self.ids.vv.text = '48.'
            self.ids.bb.text = '49.'
            self.ids.nn.text = '50.'

            listrating0 = []
            for td in gainerhtml0:
                grate0 = td.find('span')
                listrating0.append(grate0)
            lr0 = pd.DataFrame(listrating0)
            grating0 = lr0.iloc[6]
            self.ids.gainlblrt0.text = grating0.to_string()[1:]

            listrating1 = []
            for td in gainerhtml1:
                grate1 = td.find('span')
                listrating1.append(grate1)
            lr1 = pd.DataFrame(listrating1)
            grating1 = lr1.iloc[6]
            self.ids.gainlblrt1.text = grating1.to_string()[1:]

            listrating2 = []
            for td in gainerhtml2:
                grate2 = td.find('span')
                listrating2.append(grate2)
            lr2 = pd.DataFrame(listrating2)
            grating2 = lr2.iloc[6]
            self.ids.gainlblrt2.text = grating2.to_string()[1:]

            listrating3 = []
            for td in gainerhtml3:
                grate3 = td.find('span')
                listrating3.append(grate3)
            lr3 = pd.DataFrame(listrating3)
            grating3 = lr3.iloc[6]
            self.ids.gainlblrt3.text = grating3.to_string()[1:]

            listrating4 = []
            for td in gainerhtml4:
                grate4 = td.find('span')
                listrating4.append(grate4)
            lr4 = pd.DataFrame(listrating4)
            grating4 = lr4.iloc[6]
            self.ids.gainlblrt4.text = grating4.to_string()[1:]

            listrating5 = []
            for td in gainerhtml5:
                grate5 = td.find('span')
                listrating5.append(grate5)
            lr5 = pd.DataFrame(listrating5)
            grating5 = lr5.iloc[6]
            self.ids.gainlblrt5.text = grating5.to_string()[1:]

            listrating6 = []
            for td in gainerhtml6:
                grate6 = td.find('span')
                listrating6.append(grate6)
            lr6 = pd.DataFrame(listrating6)
            grating6 = lr6.iloc[6]
            self.ids.gainlblrt6.text = grating6.to_string()[1:]

            listrating7 = []
            for td in gainerhtml7:
                grate7 = td.find('span')
                listrating7.append(grate7)
            lr7 = pd.DataFrame(listrating7)
            grating7 = lr7.iloc[6]
            self.ids.gainlblrt7.text = grating7.to_string()[1:]

            listrating8 = []
            for td in gainerhtml8:
                grate8 = td.find('span')
                listrating8.append(grate8)
            lr8 = pd.DataFrame(listrating8)
            grating8 = lr8.iloc[6]
            self.ids.gainlblrt8.text = grating8.to_string()[1:]

            listrating9 = []
            for td in gainerhtml9:
                grate9 = td.find('span')
                listrating9.append(grate9)
            lr9 = pd.DataFrame(listrating9)
            grating9 = lr9.iloc[6]
            self.ids.gainlblrt9.text = grating9.to_string()[1:]

            listrating10 = []
            for td in gainerhtml10:
                grate10 = td.find('span')
                listrating10.append(grate10)
            lr10 = pd.DataFrame(listrating10)
            grating10 = lr10.iloc[6]
            self.ids.gainlblrt10.text = grating10.to_string()[1:]

            listrating11 = []
            for td in gainerhtml11:
                grate11 = td.find('span')
                listrating11.append(grate11)
            lr11 = pd.DataFrame(listrating11)
            grating11 = lr11.iloc[6]
            self.ids.gainlblrt11.text = grating11.to_string()[1:]

            listrating12 = []
            for td in gainerhtml12:
                grate12 = td.find('span')
                listrating12.append(grate12)
            lr12 = pd.DataFrame(listrating12)
            grating12 = lr12.iloc[6]
            self.ids.gainlblrt12.text = grating12.to_string()[1:]

            listrating13 = []
            for td in gainerhtml13:
                grate13 = td.find('span')
                listrating13.append(grate13)
            lr13 = pd.DataFrame(listrating13)
            grating13 = lr13.iloc[6]
            self.ids.gainlblrt13.text = grating13.to_string()[1:]

            listrating14 = []
            for td in gainerhtml14:
                grate14 = td.find('span')
                listrating14.append(grate14)
            lr14 = pd.DataFrame(listrating14)
            grating14 = lr14.iloc[6]
            self.ids.gainlblrt14.text = grating14.to_string()[1:]

            listrating15 = []
            for td in gainerhtml15:
                grate15 = td.find('span')
                listrating15.append(grate15)
            lr15 = pd.DataFrame(listrating15)
            grating15 = lr15.iloc[6]
            self.ids.gainlblrt15.text = grating15.to_string()[1:]

            listrating16 = []
            for td in gainerhtml16:
                grate16 = td.find('span')
                listrating16.append(grate16)
            lr16 = pd.DataFrame(listrating16)
            grating16 = lr16.iloc[6]
            self.ids.gainlblrt16.text = grating16.to_string()[1:]

            listrating17 = []
            for td in gainerhtml17:
                grate17 = td.find('span')
                listrating17.append(grate17)
            lr17 = pd.DataFrame(listrating17)
            grating17 = lr17.iloc[6]
            self.ids.gainlblrt17.text = grating17.to_string()[1:]

            listrating18 = []
            for td in gainerhtml18:
                grate18 = td.find('span')
                listrating18.append(grate18)
            lr18 = pd.DataFrame(listrating18)
            grating18 = lr18.iloc[6]
            self.ids.gainlblrt18.text = grating18.to_string()[1:]

            listrating19 = []
            for td in gainerhtml19:
                grate19 = td.find('span')
                listrating19.append(grate19)
            lr19 = pd.DataFrame(listrating19)
            grating19 = lr19.iloc[6]
            self.ids.gainlblrt19.text = grating19.to_string()[1:]

            listrating20 = []
            for td in gainerhtml20:
                grate20 = td.find('span')
                listrating20.append(grate20)
            lr20 = pd.DataFrame(listrating20)
            grating20 = lr20.iloc[6]
            self.ids.gainlblrt20.text = grating20.to_string()[1:]

            listrating21 = []
            for td in gainerhtml21:
                grate21 = td.find('span')
                listrating21.append(grate21)
            lr21 = pd.DataFrame(listrating21)
            grating21 = lr21.iloc[6]
            self.ids.gainlblrt21.text = grating21.to_string()[1:]

            listrating22 = []
            for td in gainerhtml22:
                grate22 = td.find('span')
                listrating22.append(grate22)
            lr22 = pd.DataFrame(listrating22)
            grating22 = lr22.iloc[6]
            self.ids.gainlblrt22.text = grating22.to_string()[1:]

            listrating23 = []
            for td in gainerhtml23:
                grate23 = td.find('span')
                listrating23.append(grate23)
            lr23 = pd.DataFrame(listrating23)
            grating23 = lr23.iloc[6]
            self.ids.gainlblrt23.text = grating23.to_string()[1:]

            listrating24 = []
            for td in gainerhtml24:
                grate24 = td.find('span')
                listrating24.append(grate24)
            lr24 = pd.DataFrame(listrating24)
            grating24 = lr24.iloc[6]
            self.ids.gainlblrt24.text = grating24.to_string()[1:]

            listrating25 = []
            for td in gainerhtml25:
                grate25 = td.find('span')
                listrating25.append(grate25)
            lr25 = pd.DataFrame(listrating25)
            grating25 = lr25.iloc[6]
            self.ids.gainlblrt25.text = grating25.to_string()[1:]

            listrating26 = []
            for td in gainerhtml26:
                grate26 = td.find('span')
                listrating26.append(grate26)
            lr26 = pd.DataFrame(listrating26)
            grating26 = lr26.iloc[6]
            self.ids.gainlblrt26.text = grating26.to_string()[1:]

            listrating27 = []
            for td in gainerhtml27:
                grate27 = td.find('span')
                listrating27.append(grate27)
            lr27 = pd.DataFrame(listrating27)
            grating27 = lr27.iloc[6]
            self.ids.gainlblrt27.text = grating27.to_string()[1:]

            listrating28 = []
            for td in gainerhtml28:
                grate28 = td.find('span')
                listrating28.append(grate28)
            lr28 = pd.DataFrame(listrating28)
            grating28 = lr28.iloc[6]
            self.ids.gainlblrt28.text = grating28.to_string()[1:]

            listrating29 = []
            for td in gainerhtml29:
                grate29 = td.find('span')
                listrating29.append(grate29)
            lr29 = pd.DataFrame(listrating29)
            grating29 = lr29.iloc[6]
            self.ids.gainlblrt29.text = grating29.to_string()[1:]

            listrating30 = []
            for td in gainerhtml30:
                grate30 = td.find('span')
                listrating30.append(grate30)
            lr30 = pd.DataFrame(listrating30)
            grating30 = lr30.iloc[6]
            self.ids.gainlblrt30.text = grating30.to_string()[1:]

            listrating31 = []
            for td in gainerhtml31:
                grate31 = td.find('span')
                listrating31.append(grate31)
            lr31 = pd.DataFrame(listrating31)
            grating31 = lr31.iloc[6]
            self.ids.gainlblrt31.text = grating31.to_string()[1:]

            listrating32 = []
            for td in gainerhtml32:
                grate32 = td.find('span')
                listrating32.append(grate32)
            lr32 = pd.DataFrame(listrating32)
            grating32 = lr32.iloc[6]
            self.ids.gainlblrt32.text = grating32.to_string()[1:]

            listrating33 = []
            for td in gainerhtml33:
                grate33 = td.find('span')
                listrating33.append(grate33)
            lr33 = pd.DataFrame(listrating33)
            grating33 = lr33.iloc[6]
            self.ids.gainlblrt33.text = grating33.to_string()[1:]

            listrating34 = []
            for td in gainerhtml34:
                grate34 = td.find('span')
                listrating34.append(grate34)
            lr34 = pd.DataFrame(listrating34)
            grating34 = lr34.iloc[6]
            self.ids.gainlblrt34.text = grating34.to_string()[1:]

            listrating35 = []
            for td in gainerhtml35:
                grate35 = td.find('span')
                listrating35.append(grate35)
            lr35 = pd.DataFrame(listrating35)
            grating35 = lr35.iloc[6]
            self.ids.gainlblrt35.text = grating35.to_string()[1:]

            listrating36 = []
            for td in gainerhtml36:
                grate36 = td.find('span')
                listrating36.append(grate36)
            lr36 = pd.DataFrame(listrating36)
            grating36 = lr36.iloc[6]
            self.ids.gainlblrt36.text = grating36.to_string()[1:]

            listrating37 = []
            for td in gainerhtml37:
                grate37 = td.find('span')
                listrating37.append(grate37)
            lr37 = pd.DataFrame(listrating37)
            grating37 = lr37.iloc[6]
            self.ids.gainlblrt37.text = grating37.to_string()[1:]

            listrating38 = []
            for td in gainerhtml38:
                grate38 = td.find('span')
                listrating38.append(grate38)
            lr38 = pd.DataFrame(listrating38)
            grating38 = lr38.iloc[6]
            self.ids.gainlblrt38.text = grating38.to_string()[1:]

            listrating39 = []
            for td in gainerhtml39:
                grate39 = td.find('span')
                listrating39.append(grate39)
            lr39 = pd.DataFrame(listrating39)
            grating39 = lr39.iloc[6]
            self.ids.gainlblrt39.text = grating39.to_string()[1:]

            listrating40 = []
            for td in gainerhtml40:
                grate40 = td.find('span')
                listrating40.append(grate40)
            lr40 = pd.DataFrame(listrating40)
            grating40 = lr40.iloc[6]
            self.ids.gainlblrt40.text = grating40.to_string()[1:]

            listrating41 = []
            for td in gainerhtml41:
                grate41 = td.find('span')
                listrating41.append(grate41)
            lr41 = pd.DataFrame(listrating41)
            grating41 = lr41.iloc[6]
            self.ids.gainlblrt41.text = grating41.to_string()[1:]

            listrating42 = []
            for td in gainerhtml42:
                grate42 = td.find('span')
                listrating42.append(grate42)
            lr42 = pd.DataFrame(listrating42)
            grating42 = lr42.iloc[6]
            self.ids.gainlblrt42.text = grating42.to_string()[1:]

            listrating43 = []
            for td in gainerhtml43:
                grate43 = td.find('span')
                listrating43.append(grate43)
            lr43 = pd.DataFrame(listrating43)
            grating43 = lr43.iloc[6]
            self.ids.gainlblrt43.text = grating43.to_string()[1:]

            listrating44 = []
            for td in gainerhtml44:
                grate44 = td.find('span')
                listrating44.append(grate44)
            lr44 = pd.DataFrame(listrating44)
            grating44 = lr44.iloc[6]
            self.ids.gainlblrt44.text = grating44.to_string()[1:]

            listrating45 = []
            for td in gainerhtml45:
                grate45 = td.find('span')
                listrating45.append(grate45)
            lr45 = pd.DataFrame(listrating45)
            grating45 = lr45.iloc[6]
            self.ids.gainlblrt45.text = grating45.to_string()[1:]

            listrating46 = []
            for td in gainerhtml46:
                grate46 = td.find('span')
                listrating46.append(grate46)
            lr46 = pd.DataFrame(listrating46)
            grating46 = lr46.iloc[6]
            self.ids.gainlblrt46.text = grating46.to_string()[1:]

            listrating47 = []
            for td in gainerhtml47:
                grate47 = td.find('span')
                listrating47.append(grate47)
            lr47 = pd.DataFrame(listrating47)
            grating47 = lr47.iloc[6]
            self.ids.gainlblrt47.text = grating47.to_string()[1:]

            listrating48 = []
            for td in gainerhtml48:
                grate48 = td.find('span')
                listrating48.append(grate48)
            lr48 = pd.DataFrame(listrating48)
            grating48 = lr48.iloc[6]
            self.ids.gainlblrt48.text = grating48.to_string()[1:]

            listrating49 = []
            for td in gainerhtml49:
                grate49 = td.find('span')
                listrating49.append(grate49)
            lr49 = pd.DataFrame(listrating49)
            grating49 = lr49.iloc[6]
            self.ids.gainlblrt49.text = grating49.to_string()[1:]


class hedgescreen(Screen):
    pass

class hedgeholdscreen(Screen):
    pass

class hedgesearchscreen(Screen):
    pass
    
class polscreen(Screen):
    pass

    def polref(self):
        i = 0
        url = 'https://senate-stock-watcher-data.s3-us-west-2.amazonaws.com/aggregate/all_transactions.csv'
        retrieve(url,'stockpolfile.csv')
        polcol_list = ["transaction_date", "asset_description", "asset_type", "type", "senator"]
        poldf = pd.read_csv('stockpolfile.csv', usecols=polcol_list, nrows=50)
        for line in poldf:
            i += 1
            poldate = (poldf['transaction_date'])
            self.ids.poldatelbl.text = poldate.to_string()
            polsen = (poldf['senator'])
            self.ids.polsenlbl.text = polsen.to_string()
            polname = (poldf['asset_description'])
            self.ids.polnamelbl.text = polname.to_string()
            poltype = (poldf['type'])
            self.ids.poltypelbl.text = poltype.to_string()
            polasset = (poldf['asset_type'])
            self.ids.polassetlbl.text = polasset.to_string()


class MyApp(App):

    def build(self):
        sm = ScreenManager()

        sm.add_widget(menuscreen(name='menu'))

        sm.add_widget(searchscreen(name='search'))
        sm.add_widget(curanscreen(name='curan'))
        sm.add_widget(spyscreen(name='spy'))
        sm.add_widget(listscreen(name='list'))

        sm.add_widget(hedgescreen(name='hedge'))
        sm.add_widget(hedgeholdscreen(name='hedgehold'))
        sm.add_widget(hedgesearchscreen(name='hedgesearch'))

        sm.add_widget(polscreen(name='pol'))

        return sm

if __name__ == '__main__':
    MyApp().run()

