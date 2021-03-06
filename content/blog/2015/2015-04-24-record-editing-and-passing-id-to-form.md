---
date: "2015-04-24"
draft: false
title: "Record editing and passing id to form"
tags: ["form"]
type: "blog"
slug: "record-editing-and-passing-id-to-form"
author: "Tomáš Votruba"
---

If you want to edit a record, you have to pass its id to save function. You can use `$form->addHidden()` though, but often it's better to use an action parameter.

Most important is fact, that **permissions to edit the record have to be check both in the action and the signal**. Record submit is signal as well (accessible by url).


```php
use Nette\Application\UI\Form;
use Nette\Application\BadRequestException;
use Nette\Application\ForbiddenRequestException;

class RecordPresenter extends BasePresenter
{
	/** @var object */
	private $record;

	/**
	 * Edit record
	 */
	public function actionEdit($id = 0)
	{
		// fetch record from database
		$this->record = $this->model->records->fetch($id);

		if (!$this->record) { // check if record exits
			throw new BadRequestException;

		} elseif ($this->record->userId != $this->user->id) { // check permissions to edit
			throw new ForbiddenRequestException;
		}

		$this['recordForm']->setDefaults($this->record); // set up default values
	}

	/**
	 * Form to edit record
	 */
	protected function createComponentRecordForm()
	{
		$form = new Form;
		$form->addText('myValue', 'Name:', 20, 60);
		$form->addSubmit('send', 'Send');
		$form->onSuccess[] = callback($this, 'recordUpdate');
		return $form;
	}

	/**
	 * Record update
	 * @form recordForm
	 */
	public function recordUpdate(Form $form)
	{
		if (!$this->record) { // check if the record exists
			throw new BadRequestException;
		}

		$values = $form->getValues();
		$this->model->records->update($this->record->id, $values);

		$this->flashMessage('Record updated!', 'success');
		$this->redirect('edit');
	}
}
```


Next step could be creating [separate function](  cs:vychozi-data-pro-editacni-formular) for checking both record existence and rights.


## Save & update form in one

In other word one form for multiple actions.

```php


/**
 * Form to manage record
 */
protected function createComponentRecordForm()
{
	// ...
	$form->onSuccess[] = callback($this, 'processRecordForm');
	// ...
}

/**
 * Record process
 * @form recordForm
 */
public function processRecordForm(Form $form)
{
	if ((int) $this->getParameter('id') && !$this->record) { // check if the record exists only while being edited
		throw new BadRequestException;
	}

	$values = $form->getValues();

	if($this->record) { // we're editing record
		$this->model->records->update($this->record->id, $values);
		$this->flashMessage('Record updated!', 'success');
		$this->redirect('edit');
	} else { // we're adding new record
		$this->model->records->insert($values);
		$this->flashMessage('Record created!', 'success');
		$this->redirect('default');
	}
}
```
