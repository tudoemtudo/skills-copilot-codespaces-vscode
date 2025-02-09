
{
    'name': 'Community with Qwen Integration',
    'version': '1.0',
    'summary': 'Plataforma de comunidade com integração ao Qwen 2.5-Max',
    'description': '''
        Este módulo cria uma plataforma de comunidade no Odoo com recursos como:
        - Fórum de discussão
        - Gamificação
        - Chatbot interativo usando a API do Qwen
        - Integração com GitHub
    ''',
    'author': 'Seu Nome',
    'depends': ['base', 'website', 'gamification', 'forum'],
    'data': [
        'views/community_qwen_views.xml',
        'views/website_templates.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'community_qwen/static/src/css/styles.css',
            'community_qwen/static/src/js/chatbot.js',
        ],
    },
    'installable': True,
    'application': True,
}
import requests
from odoo import models, fields, api

class CommunityMember(models.Model):
    _name = 'community.member'
    _description = 'Membros da Comunidade'

    name = fields.Char(string='Nome', required=True)
    email = fields.Char(string='Email', required=True)
    points = fields.Integer(string='Pontos', default=0)
    level = fields.Char(string='Nível', compute='_compute_level')

    @api.depends('points')
    def _compute_level(self):
        for member in self:
            if member.points >= 1000:
                member.level = 'Expert'
            elif member.points >= 500:
                member.level = 'Advanced'
            else:
                member.level = 'Beginner'

class CommunityQwenChat(models.Model):
    _name = 'community.qwen.chat'
    _description = 'Chatbot Qwen para Comunidade'

    question = fields.Text(string='Pergunta')
    answer = fields.Text(string='Resposta')

    def call_qwen_api(self):
        api_key = self.env['ir.config_parameter'].sudo().get_param('qwen.api_key')  # Armazene sua API Key nas configurações
        endpoint = 'https://api.qwen.com/v1/chat'

        try:
            response = requests.post(endpoint, json={
                "prompt": self.question,
                "max_tokens": 50,
                "temperature": 0.7
            }, headers={
                "Content-Type": "application/json",
                "Authorization": f"Bearer {api_key}"
            })

            if response.status_code == 200:
                self.answer = response.json().get("answer", "Erro ao processar a resposta.")
            else:
                self.answer = "Erro na chamada à API."
        except Exception as e:
            self.answer = str(e)

from odoo import http
from odoo.http import request

class CommunityQwenController(http.Controller):

    @http.route('/community/qwen_chat', type='json', auth='public', methods=['POST'], csrf=False)
    def qwen_chat(self, **kwargs):
        question = kwargs.get('question')
        chat = request.env['community.qwen.chat'].sudo().create({'question': question})
        chat.call_qwen_api()
        return {'answer': chat.answer}

        <odoo>
    <record id="view_community_member_form" model="ir.ui.view">
        <field name="name">community.member.form</field>
        <field name="model">community.member</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="email"/>
                        <field name="points"/>
                        <field name="level" readonly="1"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <menuitem id="menu_community_root" name="Comunidade"/>
    <menuitem id="menu_community_members" name="Membros" parent="menu_community_root" action="action_community_members"/>

    <record id="action_community_members" model="ir.actions.act_window">
        <field name="name">Membros da Comunidade</field>
        <field name="res_model">community.member</field>
        <field name="view_mode">tree,form</field>
    </record>
</odoo>

<odoo>
    <template id="qwen_chat_widget" name="Chatbot Qwen">
        <div class="qwen-chat-widget">
            <h3>Chatbot Qwen</h3>
            <input type="text" id="qwen-question" placeholder="Digite sua pergunta..."/>
            <button onclick="sendQuestion()">Enviar</button>
            <p id="qwen-answer"></p>
        </div>
    </template>
</odoo>

function sendQuestion() {
    const question = document.getElementById('qwen-question').value;
    fetch('/community/qwen_chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ question }),
    })
    .then(response => response.json())
    .then(data => {
        document.getElementById('qwen-answer').innerText = data.answer;
    })
    .catch(error => {
        console.error('Erro:', error);
    });
}

*.pyc
*.log
.idea/
.vscode/
__pycache__/
filestore/
# Odoo Community with Qwen Integration

Este módulo cria uma plataforma de comunidade no Odoo com integração ao Qwen 2.5-Max.

## Funcionalidades
- Gestão de membros
- Fórum de discussão
- Gamificação
- Chatbot interativo

## Instalação
1. Clone este repositório.
2. Adicione o caminho ao `addons_path` no `odoo.conf`.
3. Atualize a lista de aplicativos no Odoo e instale o módulo.

## Configuração
- Configure sua API Key do Qwen nas configurações do sistema (`qwen.api_key`).

- requests==2.31.0
